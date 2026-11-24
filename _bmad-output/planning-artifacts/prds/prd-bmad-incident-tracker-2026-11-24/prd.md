---
title: "PRD: Mini SOC Incident Tracker"
status: final
created: 2026-11-24
updated: 2026-11-24
---

# PRD: Mini SOC Incident Tracker

## 0. Document Purpose

This PRD turns the final Product Brief into short, testable requirements for a one-sitting BMAD learning project. It is for Imac (learner-PM) and downstream UX / architecture / stories. Vocabulary is Glossary-anchored; features carry globally numbered FRs. Implementation choices (React, Vite, CI, Pages) live in `addendum.md`, not here.

**Source:** `_bmad-output/planning-artifacts/briefs/brief-bmad-incident-tracker-2026-11-24/brief.md` (status: final).

## 1. Vision

Learning “how software gets built” fails when the sample product is too big—auth, databases, and heavy DevOps bury the lifecycle. Mini SOC Incident Tracker is a tiny browser app that lets a single learner log cybersecurity incidents, see them on a dashboard, resolve them, and filter the list—without accounts or a backend.

The deeper purpose is pedagogical: practice the full BMAD lifecycle (plan → architecture → build → test → PR → CI/CD → deploy) on a scope small enough to finish in one sitting, with a live GitHub Pages URL as proof. If it works, the repo becomes a reusable “BMAD in one sitting” demo—not a path to a production SOC.

## 2. Target User

### 2.1 Jobs To Be Done

- **Functional:** Quickly record and triage a handful of practice incidents (title, severity, open/resolved) in one place.
- **Learning:** Complete every BMAD phase against a real, shippable mini-product, and explain each step using this repo.
- **Contextual:** Work alone in a browser; no org SOC tooling or shared tenants.

### 2.2 Non-Users (v1)

- Real SOC teams, multi-analyst workflows, or anyone needing sync across devices/users.
- Learners seeking auth, backend, or SIEM-grade features.

### 2.3 Key User Journeys

*Hobby/solo dial — light form.*

- **UJ-1. Alex logs and closes a practice incident.** Alex, learning BMAD, opens the app, creates an incident with a title and High severity, sees it on the dashboard as Open, filters to High, then marks it Resolved and confirms the list updates. Refreshing the page keeps the data.

## 3. Glossary

- **Incident** — A single tracked cybersecurity event record. Has a Title, Severity, Status, stable unique id, and created timestamp. Cardinality: many Incidents in one browser.
- **Title** — Short human-readable name of an Incident, set at create time. Must be non-empty after trim.
- **Severity** — Priority of an Incident. Exact values: `Low`, `Medium`, `High`.
- **Status** — Lifecycle state of an Incident. Exact values: `Open`, `Resolved`. New Incidents start as `Open`.
- **Dashboard** — The primary screen that lists Incidents and exposes create, status change, and filters.
- **Filter** — Dashboard controls that narrow the visible Incident list by Severity and/or Status. Each dimension includes an `All` option (no constraint). Severity and Status filters combine with AND.

## 4. Features

### 4.1 Create Incident

**Description:** From the Dashboard, the learner creates an Incident by providing a Title and Severity. New Incidents start as Open. Realizes UJ-1.

**Functional Requirements:**

#### FR-1: Create Incident with Title and Severity

Learner can create an Incident by submitting a Title and a Severity (`Low` | `Medium` | `High`). Realizes UJ-1.

**Consequences (testable):**
- A successful create adds exactly one Incident to the Dashboard list.
- The new Incident’s Status is `Open`.
- The system assigns a stable unique id and a created timestamp (used for identity and sort; not required to display on the Dashboard).
- Create fails with a clear validation message if Title is empty/whitespace-only or Severity is missing/invalid.

**Out of Scope:**
- Editing Title or Severity after create.
- Deleting Incidents.

### 4.2 Dashboard List

**Description:** The Dashboard shows the learner’s Incidents so they can scan what exists. Realizes UJ-1. Default sort is newest-first by created timestamp. When zero Incidents match (or none exist), the Dashboard shows an empty-state message guiding the learner to create one.

**Functional Requirements:**

#### FR-2: View Incidents on Dashboard

Learner can view Incidents on the Dashboard without signing in.

**Consequences (testable):**
- Each visible Incident shows at least Title, Severity, and Status.
- With no Incidents stored, an empty state is shown (not a blank broken page).
- Default ordering is newest created first.

### 4.3 Change Status

**Description:** The learner toggles an Incident between Open and Resolved from the Dashboard. Realizes UJ-1.

**Functional Requirements:**

#### FR-3: Set Incident Status to Open or Resolved

Learner can change an existing Incident’s Status to `Open` or `Resolved`.

**Consequences (testable):**
- After the change, the Dashboard shows the updated Status for that Incident without requiring a full manual reload.
- Status values other than `Open` / `Resolved` are not offered.

**Out of Scope:**
- Intermediate statuses (e.g. Investigating, Closed).

### 4.4 Filter Incidents

**Description:** The learner narrows the Dashboard list by Severity and by Status. Realizes UJ-1. Each filter includes an All option. Severity and Status filters combine (AND).

**Functional Requirements:**

#### FR-4: Filter by Severity and Status

Learner can filter the Dashboard list by Severity and by Status.

**Consequences (testable):**
- Choosing a Severity value shows only Incidents with that Severity (unless All).
- Choosing a Status value shows only Incidents with that Status (unless All).
- Combined filters show only Incidents matching both selected constraints.
- Clearing back to All on both dimensions shows the full list (subject to sort).

### 4.5 Local Persistence

**Description:** Incident data survives a browser refresh using local browser storage. Realizes UJ-1. Mechanism details are in `addendum.md`.

**Functional Requirements:**

#### FR-5: Persist Incidents Across Refresh

Learner can refresh the page and still see previously created Incidents and their Statuses.

**Consequences (testable):**
- After create/status change, a full page refresh restores the same Incidents (same Title, Severity, Status).
- Persistence is per-browser/local only; no account or server sync is required or provided.

## 5. Non-Goals (Explicit)

- Not a real SOC platform, SIEM, or multi-analyst collaboration tool.
- No authentication, roles, or multi-user support.
- No backend, database, or cross-device sync.
- No edit of Title/Severity after create; no delete Incidents.
- No comments, attachments, assignment, notifications, or audit/SIEM integrations.
- No mobile-native apps.

## 6. MVP Scope

### 6.1 In Scope

- FR-1 through FR-5 as specified.
- Single-learner, browser-only use.
- GitHub repo, GitHub Actions CI, and GitHub Pages deploy for the learning pipeline (see addendum).

### 6.2 Out of Scope for MVP

- Edit Title/Severity; delete Incidents — confirmed non-goals for v1.
- Auth, multi-user, backend, cloud sync.
- Extra incident fields or workflow states beyond Glossary Status/Severity.

## 7. Success Metrics

**Primary**
- **SM-1**: All of FR-1–FR-5 demonstrable on a live GitHub Pages URL after one focused learning session. Validates FR-1–FR-5.
- **SM-2**: Page refresh retains Incidents. Validates FR-5.
- **SM-3**: Repo is on GitHub; CI runs via GitHub Actions; app is reachable on GitHub Pages.

**Secondary**
- **SM-4**: Learner can explain each BMAD lifecycle step using this repo as the concrete example.
- **SM-5**: Planning artifacts (brief, this PRD, architecture, stories) stay short enough to read in one sitting.

**Counter-metrics (do not optimize)**
- **SM-C1**: Feature count / “SOC realism” — do not add fields or workflows to look more production-like. Counterbalances SM-1.
- **SM-C2**: Document length — do not pad PRD/architecture for thoroughness theater. Counterbalances SM-5.

## 8. Open Questions

None blocking UX/architecture for v1.

Deferred (non-blocking, owner: UX/stories when detailing UI):
- Exact placement/copy of validation messages (FR-1).
- Whether id/created timestamp appear in the UI (system fields; sort/identity only required).

## 9. Assumptions Index

All Fast-path assumptions were approved 2026-11-24 and promoted into Glossary / FR consequences:

- Title required non-empty (trimmed); whitespace-only rejected.
- New Incidents get stable unique id + created timestamp; Status defaults to Open.
- Default list sort: newest-first.
- Empty Dashboard shows guidance empty state.
- Filters each include All; Severity + Status combine with AND.
