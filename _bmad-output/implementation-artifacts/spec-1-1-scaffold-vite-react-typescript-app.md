---
title: '1.1 Scaffold Vite React TypeScript App'
type: 'chore'
created: '2026-11-24'
status: 'done'
route: 'dispatch'
review_loop_iteration: 0
baseline_commit: 'NO_VCS'
context:
  - '{project-root}/_bmad-output/implementation-artifacts/epic-1-context.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The repo has planning artifacts only — no Vite React+TypeScript app — so the learner cannot run, lint, or build locally, and Epic 1.2 / Epic 2 have nothing to extend.

**Approach:** Generate the official Vite `react-ts` template via create-vite in a temporary empty directory, then copy only template files into the project root so `_bmad/`, `_bmad-output/`, and `.agents/` stay untouched. Install dependencies and confirm `lint`, `build`, and `dev` work with template scripts intact.

## Boundaries & Constraints

**Always:**
- Use official create-vite `react-ts` template content equivalent to `npm create vite@latest … -- --template react-ts` (AD-1).
- **Safe merge procedure:** scaffold into a throwaway empty directory (e.g. under the system temp dir or a disposable sibling path), then copy **only** template-owned paths into the project root.
- Preserve `_bmad/`, `_bmad-output/`, and `.agents/` bit-for-bit (no delete, move, rename, or overwrite of those trees).
- Preserve template scripts `dev` / `build` / `lint`.
- After merge, lockfile owns dependency versions; Node 20.19+ or 22.12+.
- Before and after scaffold, verify those three BMAD paths still exist.

**Never:**
- Run create-vite against `.` with `--overwrite`, `--force`, or any flag that empties the target directory.
- Delete, empty, or relocate `_bmad/`, `_bmad-output/`, or `.agents/` to make the root “empty” for create-vite.
- Scaffold directly into `.` when create-vite would refuse a non-empty directory and the only non-interactive escape is overwrite/empty.
- Hand-roll Vite/TS/React configs that diverge from the template baseline.
- Add domain/persistence/incident UI, CI workflows, or Vite `base` for Pages (Stories 1.2 / Epic 2).
- Put the lasting app in a subdirectory (e.g. `app/`) — structural seed expects `src/` at repo root (temp dir is for generation only).

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| Safe scaffold + merge | Root has `_bmad/`, `_bmad-output/`, `.agents/`; no app `package.json`/`src/` | create-vite runs in empty temp dir; only template files copied to root; BMAD dirs unchanged; `npm install` succeeds | If create-vite on `.` refuses non-empty: do **not** retry with overwrite — use temp+merge instead |
| Accidental overwrite path | Someone proposes `--overwrite` / emptyDir on root | Forbidden — abort that approach | Switch to temp+merge; never `rm` BMAD trees |
| Verify toolchain | Dependencies installed at root | `npm run lint` and `npm run build` exit 0; `dev`/`build`/`lint` scripts present | Fix only scaffold/template issues — do not invent custom tooling |
| Local run | After install | `npm run dev` starts the Vite app (default template UI is fine) | Port conflicts: note and retry on free port |
| BMAD integrity | Before and after merge | `_bmad/`, `_bmad-output/`, `.agents/` still present | If any missing after an attempt, stop and restore from backup/VCS before continuing |

</frozen-after-approval>

## Code Map

- `/Users/imac/bmad-incident-tracker/` — empty of app code today; only `_bmad/`, `_bmad-output/`, `.agents/`. Target **merge** root (not create-vite overwrite target).
- `_bmad/`, `_bmad-output/`, `.agents/` — **do not modify** during this story; pre/post existence checks required.
- Temp scaffold dir — disposable; discard after copying template files.
- Expected at root after merge (template-owned only): `package.json`, `vite.config.ts`, `tsconfig*.json`, `index.html`, `src/` (e.g. `main.tsx`, `App.tsx`), template lint/oxlint config, public assets if any, lockfile after install; map template `_gitignore` → `.gitignore` only if no root `.gitignore` yet.
- No existing app symbols to reuse. Do not create `src/domain/` or `src/persistence/` yet.

## Tasks & Acceptance

**Execution:**
- [x] Preflight -- confirm `_bmad/`, `_bmad-output/`, and `.agents/` exist at project root -- baseline for preservation
- [x] Temp dir -- run official Vite `react-ts` scaffold in an empty temporary directory (not project root) -- AD-1 without emptying root
- [x] Project root -- copy **only** template files from temp into `.` (configs, `index.html`, `src/`, etc.); do not copy over or touch BMAD paths -- safe merge
- [x] Cleanup -- remove the temporary scaffold directory -- no leftover nested app
- [x] `package.json` (and lockfile) -- `npm install` at project root; confirm `dev` / `build` / `lint` scripts remain -- toolchain ready
- [x] Verification -- `npm run lint` and `npm run build` succeed; spot-check `npm run dev` starts -- AC evidence
- [x] Postflight -- re-confirm BMAD paths still present; no CI or domain folders added; no `--overwrite` used -- boundaries held

**Acceptance Criteria:**
- Given the repo is ready for application source (BMAD folders already present), when the official Vite `react-ts` template is applied at the app root via safe temp-scaffold + merge, then `npm install`, `npm run lint`, and `npm run build` succeed
- And template scripts `dev` / `build` / `lint` remain available
- And the app runs locally via `npm run dev`
- And `_bmad/`, `_bmad-output/`, and `.agents/` remain present and were not deleted, emptied, or overwritten by scaffolding

## Implementation Notes

- 2026-11-24: Safe merge via create-vite@9.3.4 `react-ts` in `/tmp/vite-scaffold-*/scaffold-app`, then copy-only into repo root. No `--overwrite`. Temp removed.
- Verified: `npm install` / `lint` / `build` exit 0; `npm run dev` → HTTP 200 on 127.0.0.1:5173; BMAD dirs present before/after; no `.github/`, `src/domain/`, `src/persistence/`.
- `package.json` `"name"` remains `"scaffold-app"` (temp project name from create-vite); harmless for this story.
- Runtime Node was v24.x; install/lint/build/dev succeeded (spec cites 20.19+/22.12+).
- Parent verification re-ran lint/build/dev smoke; diff inventory at `/tmp/bmad-1-1-scaffold-diff-1795541467.txt` (NO_VCS baseline).
- Review patches: renamed `scaffold-app` → `bmad-incident-tracker` in `package.json` + `index.html`; added `rel="noopener noreferrer"` on six `App.tsx` blank links. Re-verified lint/build; post-patch diff `/tmp/bmad-1-1-scaffold-diff-1795541679.txt`.

## Spec Change Log

- 2026-11-24 (human review before approval): Replaced “force/overwrite into `.`” with temp-dir create-vite + merge-only copy. create-vite `--overwrite` empties the target (keeps only `.git`) and would destroy BMAD trees. Banned `--overwrite`/`--force`/root emptyDir. Added pre/post BMAD integrity checks and AC for preservation.

## Review Triage Log

- high — Spec previously allowed force/non-interactive overwrite on non-empty root — evidence: create-vite `--overwrite` empties dir except `.git`; would wipe `_bmad`, `_bmad-output`, `.agents`. Amended frozen intent + tasks to temp+merge; banned overwrite. KEEP: AD-1 official `react-ts` template; app files end at repo root.
- low — `package.json`/`index.html` still named `scaffold-app` — evidence: leftover temp project id from safe-merge; patched to `bmad-incident-tracker` (route: patch).
- false — Stock Vite demo App lacks incident UI — evidence: frozen Never forbids domain/incident UI this story; I/O says default template UI is fine.
- false — tsconfigs omit `strict` — evidence: shipped create-vite react-ts baseline; Always forbids hand-rolling configs that diverge from template.
- low — `target="_blank"` without `rel="noopener noreferrer"` in `src/App.tsx` — evidence: real on all six demo anchors; patched (route: patch).
- false — `.gitignore` references missing `.vscode/extensions.json` — evidence: official template pattern; not a runtime defect.
- false — No in-tree `vite-env.d.ts` — evidence: `"types": ["vite/client"]` + successful `npm run build`.
- false — Missing description/license/engines/test script — evidence: out of intent; test runner deferred to later story; AC only requires lint/build/dev.
- false — README content not in review diff — evidence: README.md exists as template boilerplate; verify path was process-only.
- false — Empty hero `alt` / `#spacer` a11y — evidence: stock template UI explicitly accepted by frozen I/O.
- false — Dark-mode vs favicon chrome mismatch — evidence: stock template styling; not product chrome this story.
- false — `getElementById('root')` null before `createRoot` — evidence: `index.html` ships `#root`; null is unreachable under template; throw-if-missing would be loud fail for unreachable state.
- false — verification-gap layer: no gaps — evidence: lint/build/dev verification matches chore intent; no test suite by design.

## Verification

**Commands:**
- `test -d _bmad && test -d _bmad-output && test -d .agents` -- expected: all exist before and after scaffold
- `npm install` -- expected: lockfile written; exit 0
- `npm run lint` -- expected: exit 0
- `npm run build` -- expected: exit 0; `dist/` produced
- `npm run dev` -- expected: Vite dev server starts (smoke; may stop after confirming)

**Manual checks (if no CLI):**
- Confirm scaffolding never used `--overwrite` / root wipe
- Confirm no nested leftover temp app directory remains in the repo
