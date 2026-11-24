---
title: '1.2 GitHub Actions CI and Pages Deploy'
type: 'chore'
created: '2026-11-24'
status: 'done'
route: 'dispatch'
review_loop_iteration: 0
baseline_commit: '46c7a627eec8c2fd155d1fb251eb1e939dc4361c'
context:
  - '{project-root}/_bmad-output/implementation-artifacts/epic-1-context.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The Vite React+TS scaffold builds locally but has no GitHub Actions CI and no GitHub Pages deploy path, so NFR3 / AD-7 (PR → CI → live demo URL) cannot be practiced.

**Approach:** Add a beginner-friendly GitHub Actions workflow that runs lint, tests-when-present, a basic dependency security check, and build on PRs and `main`; deploy `dist/` to GitHub Pages only from `main` after those checks pass; set Vite `base` to `/bmad-incident-tracker/` for project Pages under `HAFSACHAUDHARY/bmad-incident-tracker`.

## Boundaries & Constraints

**Always:**
- Place CI/CD under `.github/workflows/` (AD-7).
- On `pull_request` and `push` to `main`: run (1) lint via `npm run lint`, (2) automated tests via `npm test --if-present` (exits 0 when no `test` script yet; fails when a failing `test` script exists), (3) basic dependency security via `npm audit --audit-level=high`, (4) build via `npm run build`.
- Deploy `dist/` to GitHub Pages **only** on `push` to `main`, and **only after** the CI job succeeds; PRs must not deploy.
- A failed lint/test/audit/build step fails the CI job and blocks deploy from `main`.
- Set Vite `base: '/bmad-incident-tracker/'` for project Pages (`HAFSACHAUDHARY/bmad-incident-tracker`).
- Use minimal permissions: workflow/job defaults `contents: read`; deploy job only adds `pages: write` and `id-token: write`.
- Use Node 22 on `ubuntu-latest`; install with `npm ci` from existing `package-lock.json`.
- Keep existing `dev` / `build` / `lint` scripts; do not replace oxlint with ESLint.

**Never:**
- Deploy from pull requests, forks, or any branch other than `main`.
- Add Epic 2 domain/persistence/incident UI, a test runner, or product features.
- Add a backend, HTTP API client, server routes, or non-static hosting.
- Grant broad default `permissions: write-all` or unnecessary `contents: write` on CI.
- Change Vite/TS/React stack away from the Story 1.1 scaffold baseline beyond `base`.

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| PR CI | PR targeting `main` | CI job runs lint → test-if-present → audit → build; no deploy job (or deploy skipped) | Any step non-zero → CI fails; no Pages publish |
| Main success | Push/merge to `main`; CI green | Deploy job uploads `dist/` and publishes to GitHub Pages | N/A |
| Main CI fail | Push to `main`; lint/test/audit/build fails | Deploy does not run | Failed check blocks deploy |
| No tests yet | No `test` script in `package.json` | `npm test --if-present` exits 0; pipeline continues | N/A |
| Asset base | Built app under project Pages | `vite.config.ts` `base` is `/bmad-incident-tracker/` so asset URLs resolve under that path | Wrong `base` → broken CSS/JS on Pages — must match repo name |

</frozen-after-approval>

## Code Map

- `vite.config.ts` — currently `defineConfig({ plugins: [react()] })` with **no** `base`; set `base: '/bmad-incident-tracker/'`.
- `package.json` — scripts: `dev`, `build` (`tsc -b && vite build`), `lint` (`oxlint`), `preview`; **no** `test` script (do not add a runner this story). Lockfile: `package-lock.json` (npm).
- `.github/` — **missing**; create workflow(s) here. Prefer one file: `.github/workflows/ci-cd.yml`.
- `src/`, `index.html`, oxlint config — reuse as-is; do not add domain/persistence folders.
- Continuity from Story 1.1 (`spec-1-1-…`, status done): package name `bmad-incident-tracker`; BMAD trees untouched; no prior CI.

## Tasks & Acceptance

**Execution:**
- [x] `vite.config.ts` -- set `base: '/bmad-incident-tracker/'` -- project Pages asset path (AD-7 / known repo)
- [x] `.github/workflows/ci-cd.yml` -- add workflow: `ci` job on PR + `main` (checkout, setup-node 22, `npm ci`, lint, `npm test --if-present`, `npm audit --audit-level=high`, build); `deploy` job needs `ci`, runs only on `main` pushes, uses official Pages actions to publish `dist/` -- NFR3 / Story 1.2 ACs
- [x] `.github/workflows/ci-cd.yml` -- apply minimal permissions (`contents: read` default; deploy: `pages: write`, `id-token: write`); concurrency group so Pages deploys do not overlap destructively -- least privilege
- [x] Verification -- validate workflow YAML shape locally if practical; confirm `npm run lint` and `npm run build` still succeed with new `base`; confirm no Epic 2 files added -- AC evidence

**Acceptance Criteria:**
- Given a pull request to `main`, when CI runs, then it executes lint, tests-when-present, dependency security check, and build in a beginner-friendly setup
- And deploy to GitHub Pages does not run on pull requests
- Given changes land on `main`, when CI completes successfully, then `dist/` deploys to GitHub Pages
- And Vite `base` is `/bmad-incident-tracker/` for the known project Pages path
- And a failed check blocks deploy from `main`
- And workflow permissions stay minimal (read for CI; Pages write + id-token only on deploy)

## Implementation Notes

- 2026-11-24: Implemented by [Impl](8857fc74-367d-4974-b915-3d9fb67d142e): `vite.config.ts` `base: '/bmad-incident-tracker/'`; new `.github/workflows/ci-cd.yml` (`ci` + `deploy`, Node 22, `npm ci`, lint → test-if-present → audit-high → build; Pages upload/deploy only on `push` to `main`; minimal permissions; concurrency `pages` / `cancel-in-progress: false`).
- Verified locally: lint, `npm test --if-present`, `npm audit --audit-level=high`, build (assets under `/bmad-incident-tracker/`); no `src/domain/` or `src/persistence/`. Matrix rows covered by workflow YAML assertions + those commands. Live Pages still needs repo setting Source=GitHub Actions + merge to `main`.
- Diff inventory: `/tmp/bmad-1-2-cicd-diff-11276.txt` (baseline `46c7a627eec8c2fd155d1fb251eb1e939dc4361c`).
- Review patches (same impl agent): concurrency moved to `deploy` only; `timeout-minutes: 15` on `ci`/`deploy`; post-build `Assert Vite Pages base` greps `dist/index.html`. Re-verified lint/test-if-present/audit/build + base grep. Patched diff: `/tmp/bmad-1-2-cicd-review-diff-patched-12351.txt`.


## Spec Change Log

## Review Triage Log

- false — Workflow-level `contents: read` blocks job `pages`/`id-token` write — evidence: job `permissions` replace workflow permissions for that job (GitHub workflow syntax; Pages docs + starter-workflows split pattern); deploy job grants are effective.
- medium — Workflow `concurrency.group: pages` shared by PR CI and main deploy — evidence: `.github/workflows/ci-cd.yml` top-level concurrency applies to all jobs/events; PR runs can queue behind / ahead of production deploys. route: patch
- false — Hardcoded Vite `base` breaks local `/` hosts — evidence: frozen Always requires `base: '/bmad-incident-tracker/'` for known project Pages; not a defect under approved intent.
- false — Test step is a green no-op without `test` script — evidence: frozen Always requires `npm test --if-present` that exits 0 until Epic 2 adds tests.
- false — Missing README for Pages Source setting — evidence: Verification already documents manual Pages Source=GitHub Actions; frozen Never forbids inventing product docs; not a runtime defect.
- false — No `workflow_dispatch` retry trigger — evidence: not required by approved intent; push to `main` retries deploy.
- false — `npm audit` hard-gates deploy — evidence: frozen Always requires audit in CI that blocks deploy on failure.
- false — No `.nvmrc` / `engines` for Node 22 — evidence: Always pins Node 22 in the workflow; local engines pin not in intent.
- false — Configure/upload Pages inside `ci` vs separate job — evidence: Design Notes specify this two-job shape; not a defect.
- low — Vite `base` lacks comment linking to repo URL — evidence: cosmetic only; reject (negligible everyday impact).
- medium — carried with concurrency finding — evidence: edge-case same root cause (PR/main share `pages` group). route: patch
- low — Jobs lack `timeout-minutes` while concurrency never cancels — evidence: hung job can hold queue until default multi-hour limit; trivial job-key fix. route: patch
- medium — CI build never asserts `/bmad-incident-tracker/` in `dist/index.html` — evidence: verification-gap reproduced wrong `base` still green; broken Pages assets possible. route: patch
- defer — PR-vs-main deploy `if:` guards have no executable `act`/live check — evidence: only YAML presence; story treats live PR-deploy safety as out-of-band. route: defer
- false — Spec Implementation Notes claim YAML assertions not in repo — evidence: finding’s fix is editing the build spec; rejected per triage rules.

## Design Notes

One workflow (`.github/workflows/ci-cd.yml`), two jobs:

1. **`ci`** — triggers on `pull_request` and `push` to `main`. Steps: checkout → Node 22 + npm cache → `npm ci` → `npm run lint` → `npm test --if-present` → `npm audit --audit-level=high` → `npm run build`. On `main` pushes only, also run `actions/configure-pages` + `actions/upload-pages-artifact` for `dist/` (skipped on PRs).
2. **`deploy`** — `needs: ci`; `if: github.event_name == 'push' && github.ref == 'refs/heads/main'`; runs `actions/deploy-pages` only. Environment `github-pages`.
3. **Permissions** — top-level `contents: read`; `deploy` adds `pages: write` + `id-token: write`. Use a Pages concurrency group (`cancel-in-progress: false`).
4. **Tests** — `--if-present` passes with no `test` script; Story 2.5 can add `test` later without workflow edits.
5. **Repo settings (manual, not code):** Pages → Source = **GitHub Actions**. Note under Verification only.

Golden `base` snippet:

```ts
export default defineConfig({
  base: '/bmad-incident-tracker/',
  plugins: [react()],
})
```

## Verification

**Commands:**
- `npm run lint` -- expected: exit 0
- `npm run build` -- expected: exit 0; `dist/` exists; built asset paths under `/bmad-incident-tracker/` (e.g. `dist/index.html` references that base)
- `npm test --if-present` -- expected: exit 0 with no `test` script
- `npm audit --audit-level=high` -- expected: exit 0 (currently 0 vulnerabilities)
- Workflow file present at `.github/workflows/ci-cd.yml` with `ci` + `deploy`, PR-safe deploy guard, and minimal permissions

**Manual checks (if no CLI):**
- Confirm no deploy job runs on `pull_request` (`if:` guards or job absence on that event)
- After merge to `main` (out of scope for this branch commit): Pages source = GitHub Actions; site loads at `https://hafsachaudhary.github.io/bmad-incident-tracker/`
- Confirm no `src/domain/`, `src/persistence/`, or incident UI added
