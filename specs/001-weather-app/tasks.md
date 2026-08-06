# Tasks: Weather App

**Input**: Design documents from `/specs/001-weather-app/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md), [data-model.md](./data-model.md), [contracts/open-meteo-api.md](./contracts/open-meteo-api.md), [quickstart.md](./quickstart.md)

**Tests**: Included, but scoped to what Constitution Principle IV (NON-NEGOTIABLE) actually mandates — unit conversion, 3-hour bucketing, 5-day rollup, and city-search disambiguation — plus the three component tests already named in plan.md's Project Structure (CurrentWeatherHero, CitySearch, UnitToggle). This is not a full TDD suite for every component.

**Organization**: Tasks are grouped by user story (spec.md priorities P1–P6) so each can be implemented, tested, and demoed independently.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependency on an incomplete task)
- **[Story]**: Maps the task to its user story (US1–US6) for traceability
- File paths are exact, per plan.md's Project Structure

## Path Conventions

Single front-end project at the repository root (no backend), per plan.md: `src/`, `tests/` at repo root.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization

- [ ] T001 Create the project skeleton folders per plan.md's Project Structure: `src/{api,context,hooks,utils,theme,components,assets/icons,data}`, `tests/{unit,components}`
- [ ] T002 Initialize the Vite + React project at the repo root (`package.json`, `vite.config.js`, `index.html`, `src/main.jsx`, `src/App.jsx` placeholder)
- [ ] T003 [P] Configure Vitest + React Testing Library (test script in `package.json`, `vitest.config.js`, jsdom environment, RTL setup file)
- [ ] T004 [P] Configure ESLint + Prettier for the React/Vite project
- [ ] T005 [P] Add the `motion` (motion.dev core) dependency, scoped for the two components identified in research.md §14

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared infrastructure every user story depends on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T006 [P] Implement `SelectedLocationContext` in `src/context/SelectedLocationContext.jsx`
- [ ] T007 [P] Implement `UnitPreferenceContext` (localStorage-backed, data-model.md "Unit Preference") in `src/context/UnitPreferenceContext.jsx`
- [ ] T008 [P] Implement the Open-Meteo API client (`getForecast`, `searchCities`, `reverseGeocode`) in `src/api/openMeteo.js` per contracts/open-meteo-api.md (Constitution Principle I: sole API access point)
- [ ] T009 [P] Implement `units.js` (`toDisplayTemp`, `toDisplayWind`) in `src/utils/units.js` (Constitution Principle II)
- [ ] T010 Unit test `units.js` in `tests/unit/units.test.js` (Constitution Principle IV, NON-NEGOTIABLE) — depends on T009
- [ ] T011 [P] Implement `weatherCode.js` (Open-Meteo WMO code → `{ icon, label }`) in `src/utils/weatherCode.js`
- [ ] T012 [P] Implement `formatTime.js` (location-local time formatting, never the viewer's timezone) in `src/utils/formatTime.js`
- [ ] T013 [P] Implement `conditionTheme.js` (weather condition × day/night → `--sky-*` CSS custom properties) in `src/theme/conditionTheme.js` per research.md §12
- [ ] T014 [P] Implement shared `LoadingState`, `EmptyState`, `ErrorState` components in `src/components/LoadingState.jsx`, `EmptyState.jsx`, `ErrorState.jsx` (Constitution Principle III)
- [ ] T015 Author the hand-drawn SVG weather icon set and `WeatherIcon.jsx` in `src/assets/icons/` and `src/components/WeatherIcon.jsx` (research.md §11) — depends on T011
- [ ] T016 Implement the raw `useWeatherData(location)` fetch hook in `src/hooks/useForecast.js`, returning `{ current, hourly, daily, error, isLoading }` from a single `openMeteo.getForecast` call (Constitution Principle I) — depends on T008
- [ ] T017 Wire the `App.jsx` shell: mount both context providers, apply the base condition theme, lay out the bento-grid section containers, in `src/App.jsx` and `src/main.jsx` — depends on T006, T007, T013, T016

**Checkpoint**: Foundation ready — user story implementation can now begin.

---

## Phase 3: User Story 1 - View Current Weather for a Location (Priority: P1) 🎯 MVP

**Goal**: On load, show current temperature, location, local time, wind, today's high/low, and status for a resolved location (geolocation or fallback).

**Independent Test**: Open the app fresh; verify all six data points render for one location without any configuration (quickstart.md Scenario 1).

- [ ] T018 [P] [US1] Implement `useGeolocation` hook (permission request, 5s timeout) in `src/hooks/useGeolocation.js`
- [ ] T019 [US1] Implement location-bootstrap logic (geolocation → reverse geocode → last-used location in localStorage → default "London, GB") into `SelectedLocationContext` in `src/context/SelectedLocationContext.jsx` — depends on T006, T008, T018
- [ ] T020 [US1] Implement `useCurrentWeather` hook in `src/hooks/useCurrentWeather.js`, composing `useWeatherData` with the resolved location and deriving today's high/low from the daily block — depends on T016, T019
- [ ] T021 [US1] Implement `CurrentWeatherHero` (Fraunces hero figure; temperature, location, local time, wind, high/low, status; loading/error/empty states) in `src/components/CurrentWeatherHero.jsx` — depends on T020, T012, T011, T009, T014
- [ ] T022 [US1] Wire `CurrentWeatherHero` and its condition theme into the `App.jsx` hero slot in `src/App.jsx` — depends on T021, T013, T017
- [ ] T023 [P] [US1] Component test for `CurrentWeatherHero` in `tests/components/CurrentWeatherHero.test.jsx` (loading, success, error)
- [ ] T024 [US1] Manual validation: run quickstart.md Scenario 1 end-to-end

**Checkpoint**: MVP functional and independently demoable.

---

## Phase 4: User Story 2 - Search for a City (Priority: P2)

**Goal**: Search a city by name and select it to replace the current weather view.

**Independent Test**: Search a valid city and select it; search a duplicate-named city and confirm disambiguation; search nonsense and confirm the empty state (quickstart.md Scenario 2).

- [ ] T025 [P] [US2] Implement the `formatLocationLabel` pure helper (e.g. `"London, England, United Kingdom"`) in `src/utils/formatLocationLabel.js`
- [ ] T026 [P] [US2] Unit test `formatLocationLabel` in `tests/unit/citySearch.test.js` (Constitution Principle IV, NON-NEGOTIABLE — disambiguation logic)
- [ ] T027 [US2] Implement `useCitySearch` hook (300ms debounce, `openMeteo.searchCities`, top 5 results) in `src/hooks/useCitySearch.js` — depends on T008, T025
- [ ] T028 [US2] Implement `CitySearch` (input + glass results panel with spring open/close per research.md §14, empty state) in `src/components/CitySearch.jsx` — depends on T027, T014
- [ ] T029 [US2] Wire city selection into `SelectedLocationContext` + localStorage persistence in `src/context/SelectedLocationContext.jsx` — depends on T028, T019
- [ ] T030 [P] [US2] Component test for `CitySearch` in `tests/components/CitySearch.test.jsx` (select result, no-results state, same-named-city disambiguation)
- [ ] T031 [US2] Manual validation: run quickstart.md Scenario 2

**Checkpoint**: US1 and US2 both work independently.

---

## Phase 5: User Story 3 - View 24-Hour Forecast (Priority: P3)

**Goal**: Show 8 temperature points at 3-hour intervals for the selected location.

**Independent Test**: Select a location and confirm 8 points, each 3 hours apart, update when the location changes (quickstart.md Scenario 3).

- [ ] T032 [P] [US3] Implement `forecastBuckets.js` (1-hour series → 8 points at 3-hour intervals) in `src/utils/forecastBuckets.js`
- [ ] T033 [US3] Unit test `forecastBuckets.js` in `tests/unit/forecastBuckets.test.js` (Constitution Principle IV, NON-NEGOTIABLE) — depends on T032
- [ ] T034 [US3] Implement `HourlyForecastStrip` (inline-SVG sparkline, direct-labeled points, scroll-snap strip per research.md §13–§14) in `src/components/HourlyForecastStrip.jsx` — depends on T016, T032, T011, T009
- [ ] T035 [US3] Wire `HourlyForecastStrip` into the `App.jsx` bento layout, reactive to `SelectedLocationContext`, in `src/App.jsx` — depends on T034, T017
- [ ] T036 [US3] Manual validation: run quickstart.md Scenario 3

**Checkpoint**: US1–US3 all work independently.

---

## Phase 6: User Story 4 - View 5-Day Forecast (Priority: P4)

**Goal**: Show 5 days, each with status and low/high, for the selected location.

**Independent Test**: Select a location and confirm exactly 5 distinct days, each with a status and low/high (quickstart.md Scenario 4).

- [ ] T037 [P] [US4] Implement the 5-day rollup mapper (daily arrays → `DailyForecastEntry[]`) in `src/utils/dailyRollup.js`
- [ ] T038 [US4] Unit test the daily rollup mapper in `tests/unit/dailyRollup.test.js` (Constitution Principle IV, NON-NEGOTIABLE) — depends on T037
- [ ] T039 [US4] Implement `DailyForecastList` (dumbbell/range-bar chart, one shared temperature axis, per research.md §13) in `src/components/DailyForecastList.jsx` — depends on T016, T037, T011, T009
- [ ] T040 [US4] Wire `DailyForecastList` into the `App.jsx` bento layout in `src/App.jsx` — depends on T039, T017
- [ ] T041 [US4] Manual validation: run quickstart.md Scenario 4

**Checkpoint**: US1–US4 all work independently.

---

## Phase 7: User Story 5 - Browse Summary Weather for Major Cities (Priority: P5)

**Goal**: Show current conditions for a curated set of major cities without searching.

**Independent Test**: View the app without searching and confirm several cities' conditions are visible; selecting one shows its full detail (quickstart.md Scenario 5).

- [ ] T042 [P] [US5] Create the curated major-cities dataset (5–8 cities) in `src/data/majorCities.js`
- [ ] T043 [US5] Implement `useMajorCitiesWeather` (parallel current-conditions fetch per curated city) in `src/hooks/useMajorCitiesWeather.js` — depends on T008, T042
- [ ] T044 [US5] Implement `MajorCitiesGrid` (mosaic tiles; tap promotes a city to `SelectedLocationContext`, data-model.md relationship) in `src/components/MajorCitiesGrid.jsx` — depends on T043, T011, T009, T006
- [ ] T045 [US5] Wire `MajorCitiesGrid` into the `App.jsx` bento layout in `src/App.jsx` — depends on T044, T017
- [ ] T046 [US5] Manual validation: run quickstart.md Scenario 5

**Checkpoint**: US1–US5 all work independently.

---

## Phase 8: User Story 6 - Switch Between Celsius and Fahrenheit (Priority: P6)

**Goal**: Toggle °C/°F and have every displayed temperature update consistently and persist.

**Independent Test**: Toggle the unit and confirm every visible temperature updates immediately with no reload, and the choice persists across a location change (quickstart.md Scenario 6).

- [ ] T047 [US6] Implement `UnitToggle` (physical two-position switch, critically-damped spring via the `motion` core engine, `prefers-reduced-motion` cross-fade fallback, per research.md §14) in `src/components/UnitToggle.jsx` — depends on T007, T005
- [ ] T048 [US6] Wire `UnitToggle` into the `App.jsx` chrome in `src/App.jsx` — depends on T047, T017
- [ ] T049 [P] [US6] Component test for `UnitToggle` in `tests/components/UnitToggle.test.jsx` (toggles unit, persists across a location change)
- [ ] T050 [US6] Manual validation: run quickstart.md Scenario 6

**Checkpoint**: All six user stories are independently functional.

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that span multiple user stories

- [ ] T051 [P] Implement a global offline/network-error banner beyond the per-view error states in `src/components/OfflineBanner.jsx` (Constitution Principle III; quickstart.md Scenario 7)
- [ ] T052 [P] Accessibility pass: verify `prefers-reduced-motion`, `prefers-reduced-transparency`, and `prefers-contrast` fallbacks on `UnitToggle` and `CitySearch` (research.md §14; quickstart.md Scenario 8)
- [ ] T053 [P] Responsive pass: verify the bento grid and the hourly scroll-snap strip at mobile viewport widths
- [ ] T054 Run the full quickstart.md manual validation (all 8 scenarios) as a final regression pass
- [ ] T055 Deploy to Vercel: connect the GitHub repository, configure the production build, obtain the public Demo URL (FR-012, SC-006)
- [ ] T056 Update README.md's "Live demo" link with the deployed Demo URL and confirm the Repository URL for submission

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories
- **User Stories (Phases 3–8)**: All depend on Foundational; independent of each other, may proceed in priority order (P1→P6) or in parallel if staffed
- **Polish (Phase 9)**: Depends on all desired user stories being complete

### User Story Dependencies

- **US1 (P1)**: No dependency on other stories
- **US2 (P2)**: Extends `SelectedLocationContext` (T019, from US1) but is independently testable — its own search UI and empty/disambiguation states don't require US1's hero to exist to be verified in isolation with mocked context
- **US3 (P3)**, **US4 (P4)**: Both consume the Foundational `useWeatherData` hook independently; no dependency on US1/US2
- **US5 (P5)**: Independent; reuses the Foundational API client and `SelectedLocationContext`
- **US6 (P6)**: Independent; reuses the Foundational `UnitPreferenceContext` and `units.js`

### Within Each User Story

- Constitution-mandated unit tests (T010, T026, T033, T038) should be written alongside (ideally just before) the module they cover, per Principle IV
- Utilities/hooks before components; components before wiring into `App.jsx`
- `App.jsx` is touched by every story's final wiring task (T022, T029, T035, T040, T045, T048) — implement stories sequentially if working solo, to avoid merge conflicts in that one file

### Parallel Opportunities

- All Setup tasks marked [P] (T003–T005) run in parallel
- Within Foundational, T006–T009 and T011–T014 run in parallel; T010, T015, T016, T017 wait on their listed dependency
- Once Foundational is done, US1, US3, US4, US5, and US6 have no cross-story dependency and can be built in parallel by different contributors; US2 only needs T019 from US1 to exist first
- Within each story, tasks marked [P] (e.g. T018, T023, T025/T026, T030, T032, T037, T042, T049) run in parallel with the rest of that story's batch

---

## Parallel Example: User Story 1

```bash
# After Foundational is done, launch these together:
Task: "Implement useGeolocation hook in src/hooks/useGeolocation.js"          # T018
# Then, once T019/T020 land:
Task: "Component test for CurrentWeatherHero in tests/components/CurrentWeatherHero.test.jsx"  # T023
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 (Setup) and Phase 2 (Foundational)
2. Complete Phase 3 (US1)
3. **STOP and VALIDATE**: run quickstart.md Scenario 1
4. Deploy (T055) early if you want a real Demo URL to iterate against

### Incremental Delivery

1. Setup + Foundational → foundation ready
2. US1 → validate → this is your MVP
3. US2 → US3 → US4 → US5 → US6, each validated independently via its quickstart.md scenario before moving on
4. Polish (Phase 9) once all six stories are in

### Suggested Solo Order

Given `App.jsx` is a shared integration point, implement stories in priority order (P1→P6) rather than in parallel, even though they're logically independent — this avoids repeatedly re-merging the same file.

---

## Notes

- [P] tasks touch different files with no unmet dependency
- [Story] labels map every user-story-phase task back to spec.md for traceability
- Commit after each task or logical group (see the project's commit-message style already used on this branch)
- Constitution Principle IV tests (T010, T026, T033, T038) are NON-NEGOTIABLE — do not mark their story "done" without them passing
- Avoid: vague tasks, same-file conflicts within a single parallel batch, cross-story dependencies that break independent testability
