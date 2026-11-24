---
title: "Product Brief: Mini SOC Incident Tracker"
status: final
created: 2026-11-24
updated: 2026-11-24
---

# Product Brief: Mini SOC Incident Tracker

## Executive Summary

Mini SOC Incident Tracker is a tiny browser app for logging cybersecurity incidents. Its real job is teaching: help a beginner complete the full BMAD lifecycle—planning, architecture, build, test, pull requests, CI/CD, and deployment—in one sitting.

A single learner can create incidents, view them on a dashboard, mark them Open or Resolved, and filter by severity or status. Data stays in the browser (`localStorage`). No login, no backend, no multi-user features.

Stack and ship path: React + TypeScript + Vite on GitHub, with GitHub Actions CI/CD and GitHub Pages hosting.

## The Problem

“How real software gets built” is hard to learn when the sample product is too big—auth, databases, and heavy DevOps bury the lifecycle itself. Beginners either skip planning artifacts or never finish.

They need a product small enough to complete, yet real enough to practice every BMAD phase, including a PR, CI, and a public deploy.

## The Solution

Five capabilities only:

1. **Create** an incident with a title and severity (`Low` | `Medium` | `High`).
2. **View** incidents on a simple dashboard.
3. **Change status** between `Open` and `Resolved`.
4. **Filter** by severity and by status.
5. **Persist** with browser `localStorage` so a refresh keeps the list.

Ship as a static site on GitHub Pages.

## Who This Serves

**Primary user:** a developer learning BMAD who wants one end-to-end pass from brief to a live URL.

**Success for them:** they can explain each lifecycle step using this repo, and the live app shows all five features working.

## Success Criteria

- All five features work in the browser with no backend.
- Data survives a page refresh (`localStorage`).
- Repo is on GitHub; CI runs via GitHub Actions; the app is live on GitHub Pages.
- Planning → deploy fits one focused learning session.
- Artifacts (brief, PRD, architecture, stories) stay short and readable.

## Scope

### In (v1)

- Create incident: title + severity (`Low` / `Medium` / `High`)
- Dashboard list
- Status toggle: `Open` ↔ `Resolved`
- Filters: severity and status
- Browser `localStorage`
- React + TypeScript + Vite
- GitHub + GitHub Actions + GitHub Pages
- No authentication; single local browser user

### Out (v1)

- Login, roles, multi-user, or cloud sync
- Backend or database
- Edit title/severity after create; delete incidents
- Comments, attachments, assignment, notifications, audit logs, SIEM
- Mobile-native apps

## Learning Vision

A reusable “BMAD in one sitting” demo: clear artifacts, a tiny app, and a GitHub Pages URL that proves the pipeline—not a path to a production SOC platform.
