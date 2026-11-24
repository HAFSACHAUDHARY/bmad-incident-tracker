# Architecture Spine Rubric Review — Mini SOC Incident Tracker

**Reviewer:** good-spine checklist (rubric walker)  
**Artifact:** `ARCHITECTURE-SPINE.md`  
**Stakes:** hobby/solo learning SPA — honest judgment, proportionate severity  
**Mechanical lint:** `lint_spine.py` — 0 findings (2026-11-24)

## Gate verdict

**Pass with minor gaps.** The spine nails the main implementation divergence points for a client-only layered SPA (layering, single mutation owner, one storage key/shape, filter non-persistence, static Pages pipeline) and maps FR-1–FR-5 cleanly. Remaining issues are mostly edge behavior and one filter semantics nit—not blockers for stories on this altitude, but worth tightening before build if you want zero silent forks.

---

## Checklist walk

### 1. Fixes real divergence points for the level below; misses none material

**Adequate → strong for scope.**

AD-1–AD-9 collectively lock what two implementers would otherwise disagree on: scaffold baseline, import direction, incident list ownership, persistence surface, record schema, non-persisted filters, no backend, create validation ownership, and two empty-state cases. Structural seed and capability map give a consistent folder story.

**Gaps (not covered by AD or Deferred):**

| Gap | Why it matters |
| --- | --- |
| `localStorage` read/parse failures | Corrupt JSON or schema drift → one dev resets to `[]`, another throws and whitescreens |
| UI↔domain wiring pattern | React state/hooks vs imperative refresh—not a layer violation, but two UIs could diverge on when list updates (AD-3 implies “after mutation” but not subscription model) |

For a one-sitting learner repo, the second gap is acceptable story-level glue; the first deserves at least Deferred or a one-line AD rule.

### 2. Every AD Rule is enforceable and prevents stated divergence

**Strong overall.** Lint confirms mechanical AD shape (Binds, Prevents, Rule). Spot check:

| AD | Enforceable? | Prevents claim? |
| --- | --- | --- |
| AD-1 | Yes — inspect `package.json` / scripts vs template | Yes |
| AD-2 | Yes — import graph / lint | Yes |
| AD-3 | Yes — grep for array splice in components | Yes |
| AD-4 | Yes — single module + key constant | Yes |
| AD-5 | Yes — shared `Incident` type | Yes |
| AD-6 | Yes — no filter keys in storage | Yes |
| AD-7 | Mostly — “no fetch” and CI/Pages flow are checkable; `base` is conditional on repo name | Yes for backend sprawl |
| AD-8 | Yes — domain API tests / single validation path | Yes |
| AD-9 | Yes — UI states distinguishable in code review | Yes |

**Consistency Conventions** (“throw or return `{ ok:false, error:string }` — pick one per module”) is enforceable only after the fact; it is not backed by an AD, so two domain functions could still diverge on style. Low severity for this codebase size.

### 3. Nothing under Deferred lets two units diverge

**Mostly clean.**

Deferred items are appropriately story/UX level: component/CSS split, test runner, validation copy/placement, optional display of id/timestamp.

**Exception — low:** `vite base` / Pages owner appears in **Deferred** and in **AD-7 Rule** (“set when the repo name is known”). Until pinned, local `dev` vs production asset paths can diverge; single learner mitigates, but it is the one deferred item that touches deploy correctness.

Corrupt-storage behavior is **not** in Deferred; if intentionally open, it should be listed there or ruled in AD-4.

### 4. Named tech verified-current

**Accepted on document evidence.** Stack table pins versions with verification note (create-vite `react-ts` line, 2026-11-24). Mechanical lint accepted pins. No independent npm re-check in this review.

**Observation (informational):** Versions reflect a post-scaffold “code owns versions” stance—appropriate for AD-1.

### 5. Ratifies brownfield codebase

**N/A (greenfield).** No `src/` implementation in workspace; spine is prescriptive scaffold, not reconciliation. No finding.

### 6. Spec-driven coverage (PRD + addendum)

**Strong.**

- `binds: FR-1..FR-5` matches MVP.
- Capability → Architecture Map ties each FR to modules and ADs.
- Addendum stack (React, Vite, `localStorage`, GitHub Actions, Pages) → AD-1, AD-4, AD-7, Stack.
- SM-3 (CI + live Pages) → AD-7.
- PRD empty-state guidance → AD-9; PRD deferred validation copy/id display → spine Deferred (aligned).

**Gap — low:** FR-4 consequences require Severity **and** Status filters to combine with **AND**. AD-6 mandates non-persisted filter state and in-memory filter/sort but does not explicitly state AND semantics—implementers usually infer it; one line in AD-6 Rule would close the loop.

### 7. Parent spine inheritance

**N/A.** No parent spine referenced; no inherited ADs to weaken.

### 8. Altitude-owned dimensions: decided, deferred, or open

**Adequate.**

| Dimension | Disposition |
| --- | --- |
| Paradigm / layering | Decided (Design Paradigm + AD-2) |
| Data model | Decided (AD-5) |
| Persistence mechanism | Decided (AD-4) |
| Hosting / CI/CD | Decided (AD-7, Stack, workflows in seed) |
| Environments | Implicit single env (static Pages); sufficient at initiative hobby altitude |
| Operations (monitoring, on-call) | Correctly silent — out of scope |
| Error / resilience envelope | **Silent** — see Finding H1 |
| Testing strategy | Deferred (explicit) |
| Auth / security | Decided none (Consistency Conventions) |

Operational envelope for this product is appropriately thin; the notable silence is persistence failure modes, not missing K8s.

---

## Findings (tiered)

### High

*(None.)*

### Critical

*(None.)*

### Medium

- **M1 — Persistence load failure behavior undecided**  
  **Where:** AD-4 covers happy-path load/save; no Rule for invalid JSON, wrong shape, or `QuotaExceededError`.  
  **Risk:** Two builds diverge on empty-state vs error vs partial recovery; refresh testing (FR-5 / SM-2) becomes ambiguous.  
  **Suggested disposition:** Autofix — add one AD bullet or Deferred entry: e.g. corrupt read → treat as `[]` + optional console warn; save failures surfaced once in UI.

### Low

- **L1 — FR-4 AND semantics not repeated in AD-6**  
  **Where:** AD-6 Rule.  
  **Risk:** Misread as OR filter in a rushed story.  
  **Suggested disposition:** Autofix — “apply Severity and Status constraints together (AND).”

- **L2 — Domain error style not AD-governed**  
  **Where:** Consistency Conventions table.  
  **Risk:** Mixed throw vs `Result` in domain APIs.  
  **Suggested disposition:** Discuss or defer to first domain story; optional AD-8 extension.

- **L3 — `vite base` timing**  
  **Where:** AD-7 + Deferred.  
  **Risk:** First Pages deploy broken assets until `base` set.  
  **Suggested disposition:** Ignore for solo learner if repo name fixed at scaffold; otherwise autofix when repo exists.

- **L4 — React integration pattern open**  
  **Where:** no AD for state subscription after domain mutation.  
  **Risk:** Low for solo; two files might use inconsistent patterns.  
  **Suggested disposition:** Ignore at hobby stakes or story convention.

---

## Summary table

| Checklist item | Result |
| --- | --- |
| Divergence points for level below | Pass (one resilience gap) |
| AD Rules enforceable / preventive | Pass |
| Deferred safe | Pass (minor deploy pin timing) |
| Tech verified-current | Pass (documented) |
| Brownfield ratification | N/A |
| Spec coverage | Pass (AND wording nit) |
| Parent spine | N/A |
| Dimensions at altitude | Adequate (persistence errors silent) |

**Plus 4 low findings in this file** (L1–L4).