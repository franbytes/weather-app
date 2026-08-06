# Quickstart & Validation: Weather App

## Prerequisites

- Node.js 18+ and npm
- No API keys, no accounts, no `.env` file — Open-Meteo needs none
  (research.md §1)

## Setup

```bash
npm install
```

## Run (development)

```bash
npm run dev
```

Opens the Vite dev server (default `http://localhost:5173`). The browser will
prompt for location permission on first load (User Story 1 / FR-003) — allow
or deny it to exercise both paths.

## Test

```bash
npm run test        # Vitest: unit tests (units.js, forecastBuckets.js, city search) + component tests
```

Constitution Principle IV requires the unit-conversion, 3-hour bucketing,
5-day rollup, and city-search disambiguation tests to be green before any
task touching that logic is marked done.

## Build & preview (production)

```bash
npm run build        # outputs static assets to dist/
npm run preview      # serves the dist/ build locally for a final check
```

## Deploy

Push `dist/` (or connect the repo directly) to Vercel — zero-config for a
Vite SPA (research.md §10). Record the resulting URL as the feature's
**Demo URL**, and the GitHub repository as the **Repository URL** (FR-012,
SC-006).

## Manual validation — one scenario per user story

Run these against `npm run dev` (or the deployed Demo URL) to confirm the
feature works end-to-end. Each maps directly to spec.md's Acceptance
Scenarios.

1. **Current weather (US1)** — Open the app fresh. Within ~3s you should see:
   location name, current temperature, the location's own local time, wind
   speed, today's high/low, and a weather status with icon + label.
2. **City search (US2)** — Type a real city name (e.g. "Paris") into search,
   select a result, and confirm the whole current-weather view updates to it.
   Then search a same-named city with multiple matches (e.g. "Springfield")
   and confirm the results are distinguished by region/country. Then search a
   nonsense string and confirm a clear "no results" message appears (no blank
   screen).
3. **24-hour forecast (US3)** — With a location selected, confirm the hourly
   strip shows exactly 8 points, each 3 hours apart, each with a temperature
   and icon. Switch to a different location and confirm the strip updates.
4. **5-day forecast (US4)** — Confirm exactly 5 distinct days are listed,
   each with a weather status and a low/high range.
5. **Major cities (US5)** — Without performing any search, confirm several
   major cities' current temperature + status are visible. Select one and
   confirm it becomes the fully-detailed current-weather view (US1).
6. **Unit toggle (US6)** — Flip the °C/°F switch and confirm every visible
   temperature (hero, hourly, daily, major cities) updates immediately with
   no page reload. Change location afterward and confirm the chosen unit
   persists.
7. **Failure resilience (Principle III)** — Throttle/disable network in
   devtools and reload: confirm an explicit error state appears instead of a
   blank page or crash.
8. **Reduced motion (research.md §14)** — Enable "reduce motion" at the OS
   level and reload: confirm the unit toggle and search panel cross-fade
   instead of spring-animating, and the load-in stagger collapses to a plain
   fade.

Cross-reference: [data-model.md](./data-model.md) for the exact fields each
view reads, [contracts/open-meteo-api.md](./contracts/open-meteo-api.md) for
the API calls behind each scenario.
