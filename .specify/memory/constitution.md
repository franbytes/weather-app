<!--
Sync Impact Report
- Version change: [TEMPLATE] → 1.0.0 (initial ratification)
- Modified principles: n/a (first fill of the template)
- Added sections: Core Principles (I–V), Technology & Delivery Constraints,
  Development Workflow, Governance
- Removed sections: none
- Templates requiring review (no changes made here, per this command's scope guard):
  - .specify/templates/plan-template.md — ⚠ verify its Constitution Check gate
    references these five principles once /speckit-plan runs
  - .specify/templates/spec-template.md — ✅ no constitution-specific references
  - .specify/templates/tasks-template.md — ✅ no constitution-specific references
  - .specify/templates/checklist-template.md — ✅ no constitution-specific references
- Follow-up TODOs: none — all placeholders resolved using defaults inferred from
  the project's own feature description (a client-facing, API-driven weather app
  built as a deployable portfolio piece). Revisit and tighten once real
  constraints (chosen API, hosting target, team size) are confirmed.
-->

# Weather App Constitution

## Core Principles

### I. API-First Weather Data
All weather, forecast, and city-search data MUST be obtained through a single,
well-defined data-access layer that wraps the third-party weather API; no UI
component or view MUST call the external API directly. Rationale: centralizing
access keeps API keys, rate limits, response shaping, and error handling in one
place and makes the provider swappable without touching UI code.

### II. Consistent Units Everywhere (NON-NEGOTIABLE)
Every temperature and wind-speed value shown to the user MUST honor the user's
active unit preference (Celsius/Fahrenheit, with its paired wind-speed unit),
and unit conversion logic MUST live in exactly one shared module, never
duplicated per screen or component. Rationale: mismatched units across the
current, hourly, 5-day, and major-cities views is a highly visible and
easy-to-introduce bug class; a single source of truth prevents it.

### III. Resilient to Failure
Every network-dependent feature (city search, current weather, hourly
forecast, 5-day forecast, major-cities summary) MUST define and render a
user-facing state for "no results," "request failed," and "timed out"; the
app MUST NOT show a blank screen or an unhandled crash when the external API
misbehaves. Rationale: the app's core data comes from a third-party service
outside the team's control, so graceful degradation is part of the core UX,
not an afterthought.

### IV. Test Coverage for Core Logic (NON-NEGOTIABLE)
Unit-conversion logic, 3-hour forecast bucketing, 5-day forecast rollups, and
city-search result matching/disambiguation MUST have automated tests before
being considered done. Rationale: these are the most bug-prone, hardest to
verify by eye, and most likely to silently regress pieces of the app.

### V. Simplicity & Deployability
Choose the simplest architecture that satisfies the specification — avoid
introducing a backend, database, or state-management library unless the
feature set actually requires one. The app MUST remain deployable to a public
hosting target through a single, repeatable build step. Rationale: this is a
portfolio deliverable; shipping a working public demo matters more than
architectural sophistication.

## Technology & Delivery Constraints

- The app MUST consume a real third-party weather API for current conditions,
  hourly, and 5-day forecast data — no hardcoded or mocked weather data in the
  delivered version.
- The app MUST run client-side with, at most, a minimal backend/proxy solely
  to protect a private API key if the chosen weather API requires one; it
  MUST NOT require user accounts or authentication.
- The final deliverable MUST include a public Repository URL and a public
  Demo URL that requires no local setup to evaluate.

## Development Workflow

- Features are specified (`/speckit-specify`), planned (`/speckit-plan`), and
  broken into tasks (`/speckit-tasks`) before implementation begins, following
  the Spec Kit workflow.
- Every `/speckit-plan` run MUST include a Constitution Check confirming the
  plan complies with the five Core Principles above before implementation
  proceeds; any deviation MUST be justified in that plan's Complexity
  Tracking section.
- Constitution amendments are made by editing this file directly (solo /
  portfolio project — no formal multi-approver process required) and MUST
  update the version and Last Amended date per the policy below.

## Governance

This constitution supersedes ad-hoc practices for this project. Amendments
require: (1) editing this file, (2) recording the change in a Sync Impact
Report comment at the top of this file, and (3) bumping the version per
semantic versioning — MAJOR for backward-incompatible principle removals or
redefinitions, MINOR for new principles or materially expanded guidance,
PATCH for wording/clarification fixes. All plans and reviews MUST verify
compliance with this constitution; unjustified complexity MUST be flagged
during `/speckit-plan`.

**Version**: 1.0.0 | **Ratified**: 2026-08-06 | **Last Amended**: 2026-08-06
