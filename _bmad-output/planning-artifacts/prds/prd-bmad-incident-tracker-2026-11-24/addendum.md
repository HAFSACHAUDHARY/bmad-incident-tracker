---
title: "PRD Addendum: Mini SOC Incident Tracker"
created: 2026-11-24
updated: 2026-11-24
---

# Addendum — Technical Constraints (not FR spine)

These choices come from the Product Brief. They constrain architecture and CI; they are **not** product capabilities to restate as FRs.

## Planned stack

- **UI:** React + TypeScript
- **Tooling:** Vite
- **Persistence mechanism:** browser `localStorage` (satisfies FR-5)
- **Source control:** GitHub
- **CI/CD:** GitHub Actions
- **Hosting:** GitHub Pages (static site)

## Learning pipeline expectation

Artifacts and repo should support practicing: PR → CI → deploy to Pages, within one sitting, without a backend.
