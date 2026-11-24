---
title: "Reconciliation: Product Brief → PRD + Addendum"
created: 2026-11-24
source_brief: brief-bmad-incident-tracker-2026-11-24/brief.md
target: prd-bmad-incident-tracker-2026-11-24/prd.md + addendum.md
---

# Product Brief → PRD Reconciliation

## Covered items

| Brief theme | PRD / addendum coverage |
|-------------|-------------------------|
| Five product capabilities (create, view, status, filter, persist) | FR-1–FR-5 with testable consequences; UJ-1 exercises full flow |
| Severity `Low` / `Medium` / `High`; status `Open` / `Resolved` | Glossary + FRs; create out-of-scope matches brief (no edit/delete) |
| No auth, backend, multi-user, SIEM, mobile-native | §5 Non-Goals, §6.2 MVP out; §2.2 non-users |
| Pedagogical / one-sitting BMAD lifecycle | §1 Vision; §2.1 learning JTBD; SM-1 one session; addendum learning pipeline |
| Stack & ship path (React, TS, Vite, GitHub, Actions, Pages, `localStorage`) | `addendum.md` (explicitly not FR spine) |
| Scope exclusions (comments, attachments, assignment, etc.) | §5 Non-Goals aligned with brief “Out (v1)” |
| Short, readable artifacts | SM-3; SM-C2 counter-metric against doc bloat |
| “Not a production SOC platform” | SM-C1 counter-metric; non-goals |
| Data survives refresh | FR-5, SM-2 |
| Primary user: developer learning BMAD solo | §2 Target User, UJ-1 |

## Gaps (qualitative / dropped by FR structure)

1. **Learner success: explain the lifecycle** — Brief defines success as the learner being able to *explain each lifecycle step using this repo*. PRD metrics center on demonstrable FRs and persistence (SM-1/SM-2), not articulation or walkthrough of BMAD phases. Pedagogy is in vision/JTBD but not testable success.

2. **“The Problem” narrative** — Brief’s rationale (big sample apps bury the lifecycle; beginners skip artifacts or never finish) is absent from PRD. Downstream UX/architecture only see scope, not why *small* is a product requirement.

3. **Reusable “BMAD in one sitting” demo** — Brief Learning Vision frames the repo as a reusable teaching demo (clear artifacts + tiny app + Pages URL as pipeline proof). PRD captures anti-scope-creep (SM-C1) but not the *template/reuse* intent for future learners or instructors.

4. **Delivery success criteria vs addendum** — Brief success criteria explicitly list repo on GitHub, CI via Actions, and live Pages deploy alongside feature checks. PRD SM-1 references a “live URL”; stack/CI/Pages live only in addendum as constraints, not as named success metrics traceable to the brief checklist.

5. **Integrated executive stack mention** — Brief folds ship path into the executive summary as part of product story; split to addendum is intentional and documented in PRD §0—no content loss, but readers who only skim PRD body may miss pipeline proof until addendum (minor traceability/readability gap).

## Recommendation

**Fix first (light touch)** — Core features and exclusions reconcile well; FR spine is faithful. Before finalizing the PRD, add 1–2 sentences under Success Metrics (or a short §1.1) for: (a) learner can explain BMAD phases using the repo, (b) GitHub + CI + Pages as SM deliverables mirroring the brief. Optionally echo brief “Problem” and “Learning Vision” in Vision or Document Purpose so qualitative intent survives FR formalism. No FR changes required.

If the team accepts addendum + vision as sufficient for pipeline and pedagogy, **ok to finalize** after recording these gaps as conscious deferrals in §8 Open Questions or a one-line trace note to this file.