# Epic 1 Context: Runnable App Foundation

<!-- Compiled from planning artifacts. Edit freely. Regenerate with compile-epic-context if planning docs change. -->

## Goal

Establish a runnable Vite React + TypeScript app that builds locally and ships through a beginner-friendly GitHub Actions pipeline to GitHub Pages. This epic covers no product FRs; it unlocks the static demo URL and PR → CI → deploy practice path the rest of the project depends on.

## Stories

- Story 1.1: Scaffold Vite React TypeScript App
- Story 1.2: GitHub Actions CI and Pages Deploy

## Requirements & Constraints

- Client-only static SPA: no backend, HTTP API client, server routes, auth, or multi-user sync.
- App must be deployable as Vite `dist/` to GitHub Pages so the live demo URL can later prove FR1–FR5.
- Success target: repo on GitHub, Actions CI running, app reachable on Pages.
- Scaffold must keep template scripts (`dev` / `build` / `lint`) and succeed with `npm install`, `npm run lint`, and `npm run build`; local `npm run dev` must work.
- CI on PRs and `main` must run, in a beginner-friendly setup: (1) lint, (2) automated tests (pass with no tests until later stories add them, then enforce), (3) a basic dependency security check, and (4) build.
- Deploy to Pages runs only from `main` after those checks succeed; failed checks block deploy; PRs must not deploy.
- Node target: 20.19+ or 22.12+.

## Technical Decisions

- Scaffold from the official Vite `react-ts` template (`npm create vite@latest . -- --template react-ts`); do not hand-roll Vite/TS configs that diverge from that baseline. Stack seed matches published create-vite@9.3.4 `react-ts` (Vite ^8.3.6, React ^19.3.1, TypeScript ~7.0.2, `@vitejs/plugin-react` ^6.1.2, oxlint ^1.90.0); lockfile owns versions after scaffold.
- Static hosting only: ship `dist/`; set Vite `base` to `'/<repo-name>/'` when the GitHub repo name is known (project Pages). Concrete repo name for `base` is deferred until the repo exists.
- Place CI/CD under `.github/workflows/` (lint + checks; Pages deploy of `dist/` from the default branch).
- This epic only needs the scaffold and pipeline; layered folders (`src/domain/`, `src/persistence/`, `src/components/`) and domain/persistence behavior belong to later epics—do not invent them here beyond leaving the Vite React+TS tree ready to extend.
- Test runner choice is deferred to the first test story; CI should still have a test step that passes when no tests exist yet.

## Cross-Story Dependencies

- Story 1.2 assumes Story 1.1’s scaffold is present and buildable.
- Epic 2 (incident tracking) starts from this scaffolded app and CI/Pages path; product features are out of scope here.
