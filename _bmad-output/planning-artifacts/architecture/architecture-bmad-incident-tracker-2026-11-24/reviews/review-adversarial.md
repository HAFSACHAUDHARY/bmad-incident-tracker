---
title: Adversarial Architecture Spine Review
created: 2026-11-24
artifact: ARCHITECTURE-SPINE.md
method: >-
  For each hole, construct Unit A and Unit B (one structural level below the spine:
  typical split ownership among persistence/, domain/, and UI/App). Both units treat
  every AD as satisfied on their own terms. Integration of A-built and B-built halves
  (or two learners merging repos) yields silent incompatibility.
reviewer: adversarial (finalize gate)
---

# Adversarial Review — Architecture Spine

## Executive verdict

**CONDITIONAL PASS — 8 integration holes.** The spine’s layer model and single-key persistence intent are directionally sound, but several ADs under-specify **where** state lives, **what** bytes sit in `localStorage`, and **who** orchestrates load/save relative to domain mutation. Two compliant implementations can pass local review yet fail on merge, refresh, or pair-programming handoff.

## Method

| Step | Action |
| --- | --- |
| 1 | Enumerate AD-1..AD-9 plus Consistency Conventions as literal obligations. |
| 2 | For each obligation, ask whether a second reading still allows a different one-level-down design. |
| 3 | Pair readings into **Unit A** / **Unit B** that do not violate any AD in isolation. |
| 4 | Describe the **integration fracture** (data shape, ownership, mutation path). |
| 5 | Propose a **new or tightened AD** (or convention lock) to remove the ambiguity. |

## Holes (adversarial pairs)

### H1 — `localStorage` root JSON shape (clashing shared-data shapes)

**Governs:** AD-4, AD-5

| | Unit A (persistence) | Unit B (persistence) |
| --- | --- | --- |
| Design | `mini-soc-incidents` value is `JSON.stringify(Incident[])` — root is an array. | Same key; value is `JSON.stringify({ version: 1, incidents: Incident[] })` — root is an object wrapping the list. |
| AD compliance | One adapter module; one key; load/save the Incident list (list is the array inside). | Same argument: the list is still singular and keyed once. |

**Fracture:** A loads B’s blob → `JSON.parse` is not an array → empty dashboard or thrown error. B loads A’s blob → treats as corrupt → empty or partial migration. **FR-5** appears broken though both teams followed AD-4.

**Close with:** **AD-4a (storage envelope)** — *The value at `mini-soc-incidents` MUST be a JSON array whose elements each satisfy AD-5. No wrapper object, version field, or map keyed by id. On parse failure or non-array root, treat as empty `[]` and optionally surface a single console warning (behavior fixed, not copy).*

---

### H2 — Incident collection residence (two owners of one entity)

**Governs:** AD-3, AD-4, Structural Seed (`App.tsx` boots load)

| | Unit A (domain-centric) | Unit B (App-orchestrated) |
| --- | --- | --- |
| Design | `incidents.ts` holds module-scoped `let incidents: Incident[]`; `createIncident` / `setIncidentStatus` mutate in place; exposes `getIncidents()` / `subscribe()`. | `incidents.ts` is pure: `createIncident(list, input)` and `setIncidentStatus(list, id, status)` return **new** arrays; `App.tsx` `useState` owns the only collection. |
| AD compliance | Exactly one domain module owns the collection and exposes create + setStatus; components call those APIs. | Same — domain module exposes create + setStatus; components never splice ad hoc. |

**Fracture:** Persistence boot and save anchor differently. A saves `getIncidents()` from domain after mutation; B saves App state. Merge a domain-subscribe list UI with an App-state Dashboard → **double source of truth**: domain array and React state diverge after one status toggle; refresh shows one truth, screen another.

**Close with:** **AD-3a (collection residence)** — *The authoritative Incident collection exists in exactly one place: either (mode **S**) module state inside the sole domain owner, with React notified via subscription/context, or (mode **P**) React state in `App.tsx` (or one designated parent) with domain functions remaining pure. The spine MUST name one mode; cross-importing both patterns is forbidden.*

---

### H3 — Save orchestration boundary (conflicting state-mutation paths)

**Governs:** AD-2, AD-4, AD-3

| | Unit A (App orchestrates) | Unit B (mutation wrapper in UI) |
| --- | --- | --- |
| Design | `App.tsx`: on success from domain API, calls `save(getIncidents())` or `save(next)` explicitly. | `components/hooks/usePersistedIncidents.ts` (UI layer): wraps domain calls, calls `save` inside the hook after each successful mutation. |
| AD compliance | Load once at boot; save after every successful domain mutation. UI → Domain → Persistence direction preserved (hook lives in UI, imports persistence). | Same. |

**Fracture:** Not incompatible alone — **incompatible with H2-B + H3-A mix**: developer uses pure domain (B) but forgets save in one code path (direct `setIncidentStatus` in a child “for speed”). AD-4 does not require save to be **mechanically tied** to mutation; only a rule. Alternatively, A uses domain singleton (H2-A) while B’s hook saves App state → persist file never matches domain.

**Close with:** **AD-4a (save coupling)** — *After the spine chooses AD-3a mode: (S) domain owner invokes persistence only through a single injected `persistIncidents(incidents: Incident[])` called at the end of every successful `createIncident` / `setIncidentStatus`; (P) exactly one UI hook/wrapper in `App.tsx` performs `save` and all components MUST use it — no direct domain mutation imports from leaf components.*

---

### H4 — Domain vs persistence type dependency (clashing shared-data shapes)

**Governs:** AD-2, AD-5, Structural Seed (`types.ts`)

| | Unit A (persistence) | Unit B (persistence) |
| --- | --- | --- |
| Design | `localStore.ts` defines `StoredIncident` duplicate of AD-5 fields; maps to/from `Incident` on load/save. | `localStore.ts` imports `Incident` from `domain/types.ts` and serializes directly. |
| AD compliance | Persistence may use domain type; never imports React. Domain does not touch `localStorage`. | Same. |

**Fracture:** Learner adds optional field in A’s mapper only → saved JSON includes extra keys B never reads. TypeScript in B still says AD-5 six fields — runtime rows differ. Cross-team PR: tests pass locally, **shared-data contract** drifts from AD-5 literal.

**Close with:** **AD-5a (serialization contract)** — *Persistence MUST import `Incident` from `domain/types.ts` only (no parallel stored shape). `JSON.parse` MUST validate each element has exactly AD-5 keys with correct types; reject or drop invalid rows with deterministic rules (documented). No additional keys written on save.*

---

### H5 — Filter/sort locus vs empty-state truth (conflicting derived state)

**Governs:** AD-6, AD-9

| | Unit A (domain helper) | Unit B (UI-only filter) |
| --- | --- | --- |
| Design | `filterAndSortIncidents(incidents, filters)` in `domain/incidents.ts`; list receives **already filtered** array. | Filter state in UI; list component filters/sorts in memory; no domain helper. |
| AD compliance | Filters in UI state only; not persisted; sort newest `createdAt` first. | Same. |

**Fracture:** **AD-9** requires distinguishing global-empty vs filter-no-match. Unit A’s Dashboard checks `getIncidents().length` (global) before filtering → correct empty state. Unit B’s list checks **filtered** length only → shows “create guidance” when filters exclude all rows (violates AD-9) **or** shows “no matches” for global empty if implementation conflates checks. Both satisfy AD-6; **empty-state AD-9** depends on whether “stored Incidents” means pre- or post-filter — spine does not define.

**Close with:** **AD-9a (empty-state inputs)** — *Global empty uses `domain` collection length before filters. Filter-no-match uses count after in-memory filter/sort. The component responsible for AD-9 messaging MUST receive both counts or both arrays explicitly (API fixed in structure seed).*

---

### H6 — Create validation error surface (dual mutation paths in UI)

**Governs:** AD-8, Consistency Conventions (errors)

| | Unit A (domain) | Unit B (domain) |
| --- | --- | --- |
| Design | `createIncident` **throws** `Error` with message string on invalid title/severity. | `createIncident` returns `{ ok: true, incident } \| { ok: false, error: string }`. |
| AD compliance | Rejects empty trim and bad severity; UI surfaces domain error string; no second validation path. | Same. |

**Fracture:** Spine assigns create form + domain to different stories. Form built for throws wired to Result API → silent failure or unhandled rejection. **Convention says “pick one style per module”** — allows both modules to differ if split across `create.ts` vs `incidents.ts`. Integration is not one module.

**Close with:** **AD-8a (error contract)** — *Domain create/status APIs use a single Result style app-wide (`{ ok, ... }` OR throw). UI MUST NOT call `createIncident` except through one adapter that normalizes errors to display string.*

---

### H7 — Boot hydration vs default row materialization (clashing `createdAt` / `status`)

**Governs:** AD-4, AD-5, AD-8

| | Unit A (persistence load) | Unit B (persistence load) |
| --- | --- | --- |
| Design | On load, if row missing `status`, default `'Open'` in persistence before domain sees it. | On load, pass through JSON as-is; domain `normalizeIncident` fills defaults when read. |
| AD compliance | Every persisted Incident eventually matches AD-5; create still sets Open/UUID/ISO in domain. | Same. |

**Fracture:** Hand-edited `localStorage` or partial legacy row: A mutates on load (writes back on next save) → timestamps backfilled differently than B. **Sort order (AD-6)** and empty counts diverge. Both claim AD-5 “every Incident is …” at UI boundary but **not** at parse boundary.

**Close with:** **AD-5b (load normalization)** — *Normalization of defaults (status, id, createdAt) happens in exactly one layer named by AD-3a/4a — persistence on load OR domain on hydrate, not both and not neither.*

---

### H8 — “Single domain module” vs multiple domain files (split ownership)

**Governs:** AD-3, Consistency Conventions (naming)

| | Unit A | Unit B |
| --- | --- | --- |
| Design | All of create, setStatus, collection in `incidents.ts`. | Collection + create in `incidentStore.ts`; `setIncidentStatus` in `incidents.ts` (re-exported barrel). |
| AD compliance | One module owns collection and exposes both APIs (barrel export still “one module”). | Same narrative. |

**Fracture:** Learner implements status in `incidents.ts` calling internal array in `incidentStore.ts` while partner adds **second** in-memory cache in `incidents.ts` for “read helpers” → two mutation paths to one entity without violating AD wording. Code review does not catch without import graph rule.

**Close with:** **AD-3b (module boundary)** — *Exactly one file (`incidentStore.ts` OR `incidents.ts`, named in structure seed) may hold collection state and mutation implementations; other domain files are types/helpers only with no writable Incident array.*

---

## AD coverage stress (no new holes)

| AD | Adversarial note |
| --- | --- |
| AD-1 | Low fork risk — template pins toolchain. |
| AD-2 | H2/H3 test dependency direction; hook placement must stay explicit. |
| AD-7 | Clear ban on HTTP; no paired clash found. |
| AD-9 | Present but **underspecified input** (see H5) — pair with AD-6 filter locus. |

## Recommended spine edits (summary)

| Priority | Action |
| --- | --- |
| 1 | Add AD-4a + AD-5a + AD-5b (bytes on wire, parse/normalize locus). |
| 2 | Add AD-3a + AD-3b + AD-4a save coupling (single residence + single save path). |
| 3 | Add AD-8a + AD-9a (error contract + empty-state inputs). |
| 4 | Update Structural Seed to name AD-3a mode, sole domain state file, and whether `filterAndSort` is mandatory. |

## Verdict (canonical)

| Field | Value |
| --- | --- |
| **Verdict** | **CONDITIONAL** — adoptable after tightening AD-3, AD-4, AD-5, AD-8, AD-9 as above |
| **Holes found** | **8** (H1–H8) |
| **Artifact** | `_bmad-output/planning-artifacts/architecture/architecture-bmad-incident-tracker-2026-11-24/ARCHITECTURE-SPINE.md` |