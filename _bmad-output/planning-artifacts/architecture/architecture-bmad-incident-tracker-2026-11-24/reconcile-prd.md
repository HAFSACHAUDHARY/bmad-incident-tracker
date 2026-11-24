---
title: "PRD ↔ Architecture Spine Reconciliation"
created: 2026-11-24
updated: 2026-11-24
sources:
  - prd.md
  - addendum.md
  - ARCHITECTURE-SPINE.md
---

# Reconciliation: PRD (+ Addendum) vs Architecture Spine

## Scope

Compare **final PRD** and **addendum** to **ARCHITECTURE-SPINE.md** for:

- Quiet requirements (testable PRD consequences or addendum constraints with no spine binding)
- FR coverage holes (FR-1..FR-5 not mapped or not governed by an invariant)

## FR coverage matrix

| ID | PRD essence | Spine binding | Gap? |
| --- | --- | --- | --- |
| FR-1 | Create with Title + Severity; Open default; id + timestamp; validation on empty/invalid | AD-3, AD-5; map → domain + form | **Partial** — record shape yes; **validation rules not AD-bound** |
| FR-2 | List Title/Severity/Status; empty-state guidance; newest-first sort | AD-5, AD-6 sort; map → list | **Partial** — **empty state not in spine** |
| FR-3 | Toggle Open/Resolved; UI updates without manual reload | AD-3, AD-5 | OK (SPA paradigm implies reactivity) |
| FR-4 | Severity/Status filters, All, AND combine | AD-6 | OK |
| FR-5 | Refresh restores data; local only | AD-4, AD-7 | OK |

**Frontmatter `binds: FR-1..FR-5`** matches declared intent; two FRs lack full consequence coverage in invariants.

## Addendum ↔ spine

| Addendum item | Spine |
| --- | --- |
| React + TypeScript, Vite | AD-1, Stack table |
| localStorage | AD-4 |
| GitHub, Actions, Pages | AD-7, structural seed |
| Learning pipeline (PR → CI → deploy, one sitting) | AD-7 (lint+build on PRs, Pages deploy) | **Quiet** — not stated as explicit learning acceptance path |

## Gaps (did not land in spine)

### G1 — FR-2 empty state (testable consequence)

**PRD:** With no incidents (or none matching filters), Dashboard shows an **empty-state message guiding the learner to create one** (FR-2 consequences + §4.2).

**Spine:** Capability map points at list component only; no AD or convention for empty UI when collection is empty or filter yields zero rows.

**Risk:** Implementations ship a blank list; SM-1 / FR-2 tests fail.

**Suggested fix:** Add AD or extend FR-2 row (e.g. Dashboard must render distinguished empty state for global empty and optionally for filter-no-match).

---

### G2 — FR-1 create validation (assumptions → consequences)

**PRD:** Title non-empty after trim; whitespace-only rejected; clear validation message; invalid/missing Severity fails (FR-1 + Assumptions Index).

**Spine:** AD-5 defines shape only; Consistency Conventions mention `{ ok: false, error: string }` as optional pattern but **no rule** that create must reject invalid input before persist.

**Risk:** Persistence layer accepts bad rows; FR-1 tests fail.

**Suggested fix:** AD binding on domain `createIncident` (trim title, enum check, fail before mutate/persist).

---

### G3 — Filter-zero vs global-empty behavior

**PRD:** Empty-state applies when **zero Incidents match** (filters) **or none exist**.

**Spine:** AD-6 covers filter state and sort; silent on whether filter-no-match shares the same “create one” guidance as global empty (product may want different copy).

**Risk:** UX/stories guess; inconsistent with PRD if only global empty is implemented.

**Suggested fix:** One line in AD-6 or Deferred→resolved: both cases need user-visible empty feedback (copy detail stays UX).

---

### G4 — Success metrics as architecture checks (quiet)

**PRD:** SM-1 (all FRs on live Pages URL), SM-3 (CI + Pages reachable).

**Spine:** AD-7 defines CI/build/deploy; does not tie to **demonstrability on published Pages URL** as a build/deploy acceptance check.

**Risk:** Low — implementable from AD-7; optional spine note for learners verifying SM-3.

---

### G5 — Explicit non-goals as guardrails (quiet)

**PRD:** No edit Title/Severity, no delete, no backend/auth (§5, feature out-of-scope).

**Spine:** No negative ADs (only positive layering).

**Risk:** Low — scope discipline; optional AD “no delete/edit APIs in domain” prevents scope creep in one-sitting demo.

## Well covered (no action required)

- Incident schema, enums, defaults (AD-5)
- Single mutation owner + persistence key (AD-3, AD-4)
- Filter derivation and AND + All (AD-6)
- Stack and static-only runtime (AD-1, AD-7, addendum)
- Deferred items aligned with PRD §8 (validation placement, id/createdAt in UI)

## Verdict

**fix-first** — Close **G1** and **G2** in the spine before finalize (testable FR consequences). **G3** should be at least one explicit spine sentence. G4–G5 are optional hardening.

## Recommended spine edits (minimal)

1. New or extended AD for **FR-2 empty state** (global + filter-zero).
2. Extend **AD-3** or **AD-5** with **create validation** (trim, non-empty title, valid Severity) before persist.
3. Clarify in **AD-6** that zero visible rows after filter still require empty feedback.

After those edits, reconciliation → **ok-to-finalize**.