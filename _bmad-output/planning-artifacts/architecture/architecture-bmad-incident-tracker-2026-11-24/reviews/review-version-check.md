# Version & Reality-Check Review — Mini SOC Incident Tracker

**Reviewer lens:** Verify every committed decision was web-researched or reality-checked (not training-data assertion): library/framework versions, named technology existence/fit, and greenfield live defaults of any starter.

**Artifacts reviewed:** `ARCHITECTURE-SPINE.md`, `.memlog.md`  
**Workspace:** greenfield — no root `package.json` / scaffold not run (independent verification only)  
**Review date:** 2026-11-24 (independent re-check same calendar day as memlog)

## Verdict

**Conditional pass — stack direction is sound; version table attribution is wrong for the stated pin.**

Named platforms and the Vite `react-ts` starter still exist and fit the PRD. Node engine bands, build scripts, Oxlint-based lint, and static GitHub Pages deploy patterns match current upstream docs. However, the Stack table and memlog claim verification against **create-vite@9.3.4** `react-ts`, while the pinned versions match **vite `main` template** (or a newer floating resolution), not the **published npm tarball** for `create-vite@9.3.4`. Because AD-1 prescribes `npm create vite@latest` (currently resolving to `9.3.4`), implementers will scaffold **different** caret ranges than the spine documents unless the table is corrected or the scaffold command is pinned to a commit/tag that matches the table.

---

## Method

| Source | What was checked |
| --- | --- |
| npm registry | `create-vite@9.3.4` pack + `npm view` for latest tags |
| GitHub `vitejs/vite` | `template-react-ts/package.json` on `main` |
| npm / releases | `vite`, `react`, `@vitejs/plugin-react`, `oxlint` |
| Vite docs | static deploy + GitHub Pages `base` |
| MDN / caniuse | `crypto.randomUUID()` |
| Workspace | absent `package.json` (greenfield) |

Memlog line 13 records a `(version)` decision but **no URLs, tarball hash, or command transcript** — credibility rests on matching external sources, not an auditable artifact.

---

## Stack table vs live starter (material)

Extracted from `npm pack create-vite@9.3.4` → `package/template-react-ts/package.json`:

| Dependency | Spine / memlog | Published `create-vite@9.3.4` `react-ts` | `vite` repo `main` template |
| --- | --- | --- | --- |
| `vite` | `^8.3.8` | `^8.3.6` | `^8.3.8` |
| `@vitejs/plugin-react` | `^6.3.0` | `^6.1.2` | `^6.3.0` |
| `oxlint` | `^1.94.0` | `^1.90.0` | `^1.94.0` |
| `typescript` | `~7.0.2` | `~7.0.2` | `~7.0.2` |
| `react` / `react-dom` | `^19.3.1` | `^19.3.1` | `^19.3.1` |
| `@types/node` | *(omitted)* | `^24.19.2` | `^26.6.6` |

**Finding (high):** Spine rows for Vite, plugin-react, and Oxlint align with **GitHub `main`**, not with **npm `create-vite@9.3.4`**. Memlog explicitly ties verification to `create-vite@9.3.4` — that statement is **incorrect** for three of seven versioned rows.

**Finding (medium):** AD-1 uses `npm create vite@latest` while the Stack names `create-vite@9.3.4`. Today `@latest` **is** `9.3.4`, but `@latest` can move independently of a frozen table; the table should either (a) quote the tarball you actually inspected, or (b) say “post-scaffold `package.json` is source of truth” and drop false precision.

**Finding (low):** Omitting `@types/node`, `@types/react`, `@types/react-dom` from the Stack is acceptable for a short seed table, but `@types/node` drifted materially (`^24.19.2` in tarball vs `^26.6.6` on `main`) — worth one footnote if types matter for CI Node typings.

---

## Item-by-item technology check

### Verified — matches web / npm (2026-11-24)

| Decision | Evidence |
| --- | --- |
| `create-vite` exists; `react-ts` template listed | [npm create-vite](https://www.npmjs.com/package/create-vite); [README templates](https://github.com/vitejs/vite/blob/main/packages/create-vite/README.md) |
| Latest create-vite line `9.3.4` | `npm view create-vite version` → `9.3.4` (published 2026-11-05) |
| Vite `^8.3.8` is current stable | [npm vite](https://www.npmjs.com/package/vite) `latest` → 8.3.8 |
| React `^19.3.1` is current stable | [npm react](https://www.npmjs.com/package/react) |
| `@vitejs/plugin-react@6.3.0` exists; peers `vite ^8.0.0` | [npm @vitejs/plugin-react](https://www.npmjs.com/package/@vitejs/plugin-react) |
| TypeScript `~7.0.2` in official react-ts template | Tarball + `main` template |
| Node `20.19+` or `22.12+` | create-vite README compatibility note; Vite 8.3.8 engines |
| Scripts: `build` = `tsc -b && vite build`, `lint` = `oxlint` | Published `react-ts` template (Oxlint default since create-vite 9.1.x react templates) |
| Static `dist/` + GitHub Actions Pages | [Vite static deploy — GitHub Pages](https://vite.dev/guide/static-deploy) |
| Vite `base: '/<repo-name>/'` for project Pages site | Same Vite guide (path prefix for `https://<user>.github.io/<repo>/`) |
| `localStorage`, layered SPA, no backend | Browser/platform facts; fits PRD — not version-sensitive |
| GitHub Actions + GitHub Pages as hosting/CI | [GitHub Pages custom workflows](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages) — still supported |

### Verified with caveats — not fully web-audited in spine/memlog

| Decision | Status | Notes |
| --- | --- | --- |
| `crypto.randomUUID()` for incident `id` (AD-5, memlog assumption) | **Fit OK; constraint unstated** | Baseline in modern browsers (~97% global); **secure context** (HTTPS). GitHub Pages URL is HTTPS — fine for production. Plain HTTP preview is edge case; spine does not mention secure context. [MDN randomUUID](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID) |
| CI: “lint + build on PRs” (AD-7) | **Pattern valid; unpinned** | No specific workflow file, action major versions, or `pull_request` trigger cited — reasonable deferral to stories, but **not** version-researched in memlog |
| Filter state ephemeral; ISO `createdAt` | **Design assumptions** | Not version decisions — no web check required |
| Deferred: Vitest / test runner | **N/A** | No versions asserted |

### Not train-wreck risks (informational)

- create-vite changelog shows TypeScript 6 vs 7 toggles for **ESLint** templates; **react-ts** uses Oxlint + TS 7 — spine lint choice matches current template.
- `npm view oxlint version` → `1.95.0` while spine cites template caret `^1.94.0` — normal caret drift within range after scaffold.

---

## Documented verification vs reviewer re-check

| Claim | Memlog / spine | Independent result |
| --- | --- | --- |
| Verified 2026-11-24 against create-vite@9.3.4 react-ts | Yes | **Partially false** for vite, plugin-react, oxlint — matches `main`, not tarball |
| create-vite@9.3.4 line | Named in Stack | **Correct** as current npm latest |
| “Code owns versions after scaffold” | Stack footnote | **Good mitigation** but does not excuse wrong pre-scaffold table if referenced for planning |

---

## Recommendations (for spine / memlog maintenance)

1. **Fix attribution:** Change Stack note to “verified against `vite` `main` `template-react-ts` on 2026-11-24” **or** republish table from `npm pack create-vite@9.3.4` (accept `^8.3.6`, `^6.1.2`, `^1.90.0`).
2. **Align AD-1 with table:** Either pin scaffold (`npm create vite@9.3.4 ...`) when citing that line, or drop exact caret pins until after first scaffold commit.
3. **Memlog provenance:** Add one line with reproducible command, e.g. `npm pack create-vite@9.3.4 && tar -xOf .../package.json`.
4. **Optional AD-5 footnote:** `crypto.randomUUID()` requires secure context — satisfied on GitHub Pages; localhost dev OK.

---

## Summary table

| Severity | Count | Theme |
| --- | --- | --- |
| High | 1 | Stack versions ≠ published create-vite@9.3.4 despite explicit claim |
| Medium | 2 | `@latest` vs pinned table; no auditable memlog provenance |
| Low | 3 | Omitted @types/* ; crypto secure-context ; unpinned GHA workflow versions |

**Bottom line:** Technologies and starter choice are current and appropriate. **Do not treat the Stack caret pins as what `npm create vite@latest` will write today** until attribution is fixed or scaffold is run and `package.json` is copied into the spine.