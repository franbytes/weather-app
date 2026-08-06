# Implementation Plan: Weather App

**Branch**: `001-weather-app` | **Date**: 2026-08-06 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-weather-app/spec.md`

## Summary

A client-only React + Vite single-page app that shows current weather, an
8-point (3-hour interval) 24-hour forecast, a 5-day forecast, a curated
major-cities summary, and city search — all unit-aware (°C/°F) — sourced
entirely from the free, key-less Open-Meteo Forecast and Geocoding APIs, with
no backend, deployed as a static site. Design direction ("Almanac Console":
editorial display type + instrument-panel data type + condition-driven
gradient theming), the two required visualizations (24h sparkline, 5-day
dumbbell), and the interruptible/translucent motion treatment for the unit
toggle and city search were decided using the `frontend-design`, `dataviz`,
and `apple-design` skills; full rationale in [research.md](./research.md).

## Technical Context

**Language/Version**: JavaScript (ES2022), JSX via React 18

**Primary Dependencies**: React 18, Vite 5, and one small (~2.5kB) spring
animation engine (`motion`, motion.dev core) scoped to exactly two components
(research.md §14). No chart, state-management, or general animation/UI
library otherwise (research.md §3, §4, §13).

**Storage**: `localStorage` only (last-selected location, unit preference) — no
database, no backend.

**Testing**: Vitest + React Testing Library

**Target Platform**: Modern evergreen browsers (desktop + mobile web)

**Project Type**: Single-page front-end web application (no backend/API of
our own — the only "backend" is the third-party Open-Meteo API consumed
directly from the client)

**Performance Goals**: Initial current-weather render within 3s on a typical
broadband connection (SC-001); unit toggle re-render with no network
round-trip (SC-004)

**Constraints**: No API keys to manage (Open-Meteo requires none); must build
to static assets deployable with a single build step (Constitution Principle V)

**Scale/Scope**: Single anonymous-user session at a time (no accounts, no
multi-tenant data); ~5–8 curated major cities; 6 top-level views/sections per
spec's user stories

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-checked after Phase 1 design.*

| Principle | Check | Status |
|---|---|---|
| I. API-First Weather Data | All Open-Meteo calls (forecast + geocoding) go through one module, `src/api/openMeteo.js`; no component calls `fetch` directly against the weather API. | PASS |
| II. Consistent Units Everywhere (NON-NEGOTIABLE) | All temperature/wind values render through `src/utils/units.js`; no inline conversion anywhere else. | PASS |
| III. Resilient to Failure | Every data hook (`useCurrentWeather`, `useForecast`, `useCitySearch`) returns `{ data, error, isLoading }`; every consuming view has an explicit error/empty/loading branch (FR-005, FR-011). | PASS |
| IV. Test Coverage for Core Logic (NON-NEGOTIABLE) | Vitest unit tests planned for `units.js`, the 3-hour bucketing selector, the 5-day rollup mapper, and city-search result labeling/disambiguation, before those tasks are marked done. | PASS |
| V. Simplicity & Deployability | No backend, no state-management library, no chart library; a single ~2.5kB spring engine is the one scoped exception (see Complexity Tracking) — everything else is CSS. Single `vite build` → static deploy to Vercel. | PASS (with one documented, minimal exception) |

## Project Structure

### Documentation (this feature)

```text
specs/001-weather-app/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
index.html
vite.config.js
package.json

src/
├── main.jsx                  # Entry point
├── App.jsx                   # Top-level layout assembly (the bento grid)
├── api/
│   └── openMeteo.js           # Sole Open-Meteo client: forecast + geocoding calls (Principle I)
├── context/
│   ├── SelectedLocationContext.jsx
│   └── UnitPreferenceContext.jsx
├── hooks/
│   ├── useCurrentWeather.js
│   ├── useForecast.js         # returns both hourly (bucketed) and daily (rolled-up) shapes
│   ├── useCitySearch.js
│   └── useGeolocation.js
├── utils/
│   ├── units.js                # toDisplayTemp / toDisplayWind (Principle II)
│   ├── forecastBuckets.js      # hourly → 3-hour-interval selector (FR-006)
│   ├── weatherCode.js          # Open-Meteo WMO code → { icon, label }
│   └── formatTime.js           # location-local time formatting (Edge Case: local vs viewer TZ)
├── theme/
│   └── conditionTheme.js       # weather condition + day/night → --sky-* CSS custom properties
├── components/
│   ├── CurrentWeatherHero.jsx
│   ├── HourlyForecastStrip.jsx # the 24h sparkline chart (research.md §13); scroll-snap strip (§14)
│   ├── DailyForecastList.jsx   # the 5-day dumbbell chart (research.md §13)
│   ├── MajorCitiesGrid.jsx
│   ├── CitySearch.jsx          # spring-driven results panel, glass material (research.md §14)
│   ├── UnitToggle.jsx          # spring-driven physical switch (research.md §14)
│   ├── WeatherIcon.jsx
│   └── ErrorState.jsx / EmptyState.jsx / LoadingState.jsx  (Principle III)
└── assets/
    └── icons/                  # hand-authored SVG weather glyph set (research.md §11)

tests/
├── unit/
│   ├── units.test.js
│   ├── forecastBuckets.test.js
│   └── citySearch.test.js
└── components/
    ├── CurrentWeatherHero.test.jsx
    ├── CitySearch.test.jsx
    └── UnitToggle.test.jsx
```

**Structure Decision**: Single front-end project at the repository root (no
`frontend/`/`backend/` split — there is no backend). This matches Option 1
(single project) from the template, specialized for a front-end SPA: `src/`
is organized by responsibility (api / context / hooks / utils / theme /
components) rather than by route, since the app is a single page with several
in-page sections, not a multi-route application.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| One ~2.5kB spring animation engine (`motion` core) added as a dependency, beyond the "CSS-only" default set by Principle V | The unit toggle and city-search panel are the app's only two genuinely gesture-driven, repeatedly-tapped controls; per the `apple-design` skill, CSS `transition`/`@keyframes` cannot be smoothly re-grabbed/interrupted mid-motion, which reads as "stuck" on a control tapped often | Pure CSS transitions — rejected specifically for these two controls because interrupting a mid-flight toggle/panel animation with a CSS transition causes a visible jump/restart rather than a smooth re-target; acceptable everywhere else in the app (load stagger, hover/press), so the dependency is not used generally |

## Post-Design Constitution Check (after Phase 1)

Re-evaluated against [data-model.md](./data-model.md) and
[contracts/open-meteo-api.md](./contracts/open-meteo-api.md): no additional
dependency, backend, or state library was introduced while designing the data
model and API contracts beyond the one documented above. All five principles
still PASS (Principle V with its one documented, scoped exception). No
further updates required to the tables above.
