---
status: final
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-bmad-incident-tracker-2026-11-24/prd.md
  - _bmad-output/planning-artifacts/prds/prd-bmad-incident-tracker-2026-11-24/addendum.md
  - _bmad-output/planning-artifacts/architecture/architecture-bmad-incident-tracker-2026-11-24/ARCHITECTURE-SPINE.md
---

# bmad-incident-tracker - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for bmad-incident-tracker (Mini SOC Incident Tracker), decomposing the requirements from the PRD and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: Learner can create an Incident with a non-empty Title and Severity (`Low` | `Medium` | `High`); new Incidents are `Open` with system-assigned id and created timestamp; invalid create shows a clear validation message (PRD FR-1 / AD-5 / AD-8).
FR2: Learner can view Incidents on a Dashboard without signing in; each row shows at least Title, Severity, and Status; default order newest-first; global empty state guides create (PRD FR-2 / AD-9).
FR3: Learner can set an Incident Status to `Open` or `Resolved`; Dashboard updates without a full manual reload; no other statuses (PRD FR-3).
FR4: Learner can filter by Severity and Status (each with `All`); filters combine with AND; clearing both to All restores full sorted list; filter-no-match empty state is distinct from global empty (PRD FR-4 / AD-6 / AD-9).
FR5: After create/status change, a full page refresh restores the same Incidents (Title, Severity, Status) via local browser persistence only — no account or server sync (PRD FR-5 / AD-4).

### NonFunctional Requirements

NFR1: Client-only SPA — no backend, API client, auth, or multi-user sync (PRD non-goals / AD-7).
NFR2: Persistence is per-browser `localStorage` with key `mini-soc-incidents`, JSON array envelope; corrupt/invalid load yields empty list (AD-4).
NFR3: App must be deployable as static `dist/` to GitHub Pages with CI (lint + build on PR; deploy from default branch) so FR1–FR5 are demonstrable on a live URL (PRD SM-3 / AD-7).
NFR4: Architecture layering enforced: UI → Domain (pure) → Persistence types; Mode P — App owns Incident[] state and save orchestration (AD-2 / AD-3).
NFR5: Scope stays minimal — no edit Title/Severity, no delete, no extra SOC workflows (PRD non-goals / SM-C1).

### Additional Requirements

- **Starter (Epic 1 Story 1):** Scaffold with official Vite `react-ts` template (`npm create vite@latest . -- --template react-ts`); keep template scripts (AD-1).
- Layered folders: `src/domain/` (`types.ts`, `incidents.ts`), `src/persistence/localStore.ts`, `src/components/`, `App.tsx` owns state (Structural Seed).
- Domain Result contract: `{ ok: true, incidents } | { ok: false, error }` for create and setStatus (AD-8).
- Incident shape exactly `{ id, title, severity, status, createdAt }` (AD-5).
- Vite `base: '/<repo-name>/'` when GitHub repo name is known (project Pages) (AD-7).
- GitHub Actions (beginner-friendly): on PRs and main — lint, automated tests (when present), basic dependency security check, and build; deploy `dist/` to GitHub Pages **only from `main` after successful checks** (AD-7 / Story 1.2).
- No HTTP API / server routes (AD-7).

### UX Design Requirements

None — UX design workflow was skipped; empty-state and validation copy follow PRD/Architecture ADs only (AD-8, AD-9).

### FR Coverage Map

FR1: Epic 2 — Create Incident with Title and Severity  
FR2: Epic 2 — View Incidents on Dashboard  
FR3: Epic 2 — Set Incident Status Open/Resolved  
FR4: Epic 2 — Filter by Severity and Status  
FR5: Epic 2 — Persist Incidents across refresh  

### Epic List

### Epic 1: Runnable App Foundation
Learner has a Vite React+TypeScript app that builds locally, with GitHub Actions CI and GitHub Pages deploy path ready for the live demo URL.
**FRs covered:** (none — enables NFR3 / AD-1 / AD-7)

### Epic 2: Track & Resolve Incidents
Learner can create, list, change status, filter, and keep Incidents across refresh — full practice SOC tracking loop (UJ-1).
**FRs covered:** FR1, FR2, FR3, FR4, FR5

## Epic 1: Runnable App Foundation

Learner has a Vite React+TypeScript app that builds locally, with GitHub Actions CI and GitHub Pages deploy path ready for the live demo URL.

### Story 1.1: Scaffold Vite React TypeScript App

As a learner,
I want the project scaffolded from the official Vite `react-ts` template,
So that I start from a known baseline I can build and run locally (AD-1).

**Acceptance Criteria:**

**Given** the repo is ready for application source  
**When** I scaffold with `npm create vite@latest` using the `react-ts` template in the app root  
**Then** `npm install`, `npm run lint`, and `npm run build` succeed  
**And** template scripts `dev` / `build` / `lint` remain available  
**And** the app runs locally via `npm run dev`

### Story 1.2: GitHub Actions CI and Pages Deploy

As a learner,
I want simple CI checks and Pages deploy from `main` only after those checks pass,
So that I practice a real PR → CI → deploy pipeline (NFR3 / AD-7 / SM-3).

**Acceptance Criteria:**

**Given** the scaffolded app is on GitHub  
**When** a pull request runs CI  
**Then** the workflow runs, in a beginner-friendly setup: (1) lint, (2) automated tests (pass with no tests until Epic 2 adds them, then enforce them), (3) a basic dependency security check, and (4) build  
**And** deploy to GitHub Pages does **not** run on pull requests  

**Given** changes land on `main`  
**When** CI completes successfully (lint, tests, dependency security check, build)  
**Then** `dist/` deploys to GitHub Pages  
**And** Vite `base` is set for the project Pages path (`/<repo-name>/`) when the repo name is known  
**And** a failed check blocks deploy from `main`

## Epic 2: Track & Resolve Incidents

Learner can create, list, change status, filter, and keep Incidents across refresh — full practice SOC tracking loop (UJ-1).

### Story 2.1: Incident Domain and Local Persistence

As a learner,
I want Incident types, pure create/setStatus APIs, and `localStorage` load/save wired through App,
So that the app has one data shape and one persistence path (AD-3, AD-4, AD-5, AD-8; enables FR5).

**Acceptance Criteria:**

**Given** the scaffolded app from Epic 1  
**When** I add `src/domain/types.ts` and `src/domain/incidents.ts`  
**Then** `Incident` matches exactly `{ id, title, severity, status, createdAt }` with Glossary enums  
**And** `createIncident` / `setIncidentStatus` are pure and return `{ ok: true, incidents } | { ok: false, error }`  
**And** no React or `localStorage` imports exist under `src/domain/`

**Given** App owns the Incident list in React state (Mode P)  
**When** the app boots  
**Then** it loads once via `src/persistence/localStore.ts` using key `mini-soc-incidents`  
**And** the stored value is a JSON array (no wrapper object)  
**And** corrupt JSON, non-array root, or storage errors yield `[]` (optional single `console.warn`)

**Given** a successful domain mutation applied in App  
**When** state updates  
**Then** App saves the full Incident array through the same persistence adapter  
**And** leaf components do not call domain mutations or `localStorage` directly

### Story 2.2: Create Incident and Dashboard List

As a learner,
I want to create an Incident with title and severity and see it on the Dashboard,
So that I can log practice incidents (FR1, FR2).

**Acceptance Criteria:**

**Given** the Dashboard is open with no stored Incidents  
**When** the page loads  
**Then** a create-guidance empty state is shown (not a blank/broken page) (AD-9)

**Given** I enter a non-empty Title and choose Severity `Low`, `Medium`, or `High`  
**When** I submit create  
**Then** exactly one Incident appears on the Dashboard with Status `Open`  
**And** it shows at least Title, Severity, and Status  
**And** the list is newest-first by `createdAt`

**Given** Title is empty/whitespace-only or Severity is missing/invalid  
**When** I submit create  
**Then** create fails and the Dashboard shows the domain `error` string  
**And** no Incident is added

**Given** I created one or more Incidents  
**When** I refresh the browser  
**Then** the same Incidents remain (Title, Severity, Status) (FR5)

### Story 2.3: Change Incident Status

As a learner,
I want to set an Incident’s Status to Open or Resolved from the Dashboard,
So that I can close out practice incidents (FR3).

**Acceptance Criteria:**

**Given** an Incident is visible on the Dashboard  
**When** I set its Status to `Resolved` (or back to `Open`)  
**Then** the Dashboard shows the updated Status without a full manual page reload  
**And** only `Open` and `Resolved` are offered

**Given** a Status change succeeded  
**When** I refresh the browser  
**Then** the updated Status is still present (FR5)

**Given** an invalid status value  
**When** `setIncidentStatus` is called  
**Then** it returns `{ ok: false, error }` and App state is unchanged

### Story 2.4: Filter by Severity and Status

As a learner,
I want to filter the Dashboard by Severity and Status with All options,
So that I can narrow the practice list (FR4).

**Acceptance Criteria:**

**Given** multiple Incidents with mixed Severity and Status  
**When** I choose a Severity value (not All)  
**Then** only Incidents with that Severity are shown  
**And** Status filter defaults/behaves with All unless also set

**Given** both Severity and Status filters are set to specific values  
**When** the list renders  
**Then** filters combine with **AND** (AD-6)  
**And** filter selections are not written to `localStorage`

**Given** both filters are set back to All  
**When** the list renders  
**Then** the full list appears, newest-first

**Given** stored Incidents exist but none match the active filters  
**When** the list renders  
**Then** a distinct “no matches” empty state is shown (not the create-guidance copy) (AD-9)  
**And** the Dashboard uses both pre-filter and post-filter counts (or arrays)

### Story 2.5: Automated Tests for Core Behavior

As a learner,
I want simple automated tests for domain and persistence behavior,
So that CI (Story 1.2) runs real checks that stay beginner-friendly.

**Acceptance Criteria:**

**Given** the Epic 2 domain and persistence modules exist  
**When** I run the project’s test command in CI and locally  
**Then** tests cover at least: successful create; rejected empty title; setStatus Open↔Resolved; AND filter helper or equivalent; load/save round-trip for a JSON array under `mini-soc-incidents`  
**And** the test setup stays minimal (one mainstream runner, e.g. Vitest, without a heavy e2e suite)  
**And** Story 1.2’s CI test step fails the pipeline when these tests fail
