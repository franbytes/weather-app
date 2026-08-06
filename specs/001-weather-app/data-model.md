# Phase 1 Data Model: Weather App

Derived from spec.md's Key Entities, shaped by the Open-Meteo contract in
[contracts/open-meteo-api.md](./contracts/open-meteo-api.md). This is an
in-memory client-side model — nothing here is persisted server-side; the only
persisted fields are called out under **Unit Preference** and
**Selected Location** (`localStorage`).

## Location

Identifies a place the user can view weather for (their own detected
location, a searched city, or a curated major city).

| Field | Type | Notes |
|---|---|---|
| `id` | string | Stable key, e.g. `"{lat}:{lon}"`, for caching/dedup |
| `name` | string | City/place name |
| `admin1` | string \| null | State/region, for disambiguation (FR-005) |
| `country` | string | Country name |
| `countryCode` | string | ISO country code, for flags/labels |
| `latitude` | number | Decimal degrees |
| `longitude` | number | Decimal degrees |
| `timezone` | string | IANA timezone (e.g. `"Europe/London"`), drives the "location's own local time" rule |

**Validation**: `latitude` ∈ [-90, 90], `longitude` ∈ [-180, 180]. A search
result missing `admin1` displays as `"{name}, {country}"`.

## Unit Preference

| Field | Type | Notes |
|---|---|---|
| `unit` | `"celsius" \| "fahrenheit"` | Default `"celsius"` |

**Persistence**: `localStorage["weather-app:unit"]`. **State transition**: a
single toggle between the two values; no other states exist (FR-009, FR-010).

## Selected Location (session state)

| Field | Type | Notes |
|---|---|---|
| `location` | `Location \| null` | `null` only before first resolution (geolocation/default-city race, FR-003) |
| `source` | `"geolocation" \| "default" \| "search" \| "major-city"` | Drives no UI directly; useful for the Resilient-to-Failure error copy ("couldn't get your location, showing London") |

**Persistence**: last resolved `Location` cached in
`localStorage["weather-app:last-location"]` and used as the default on repeat
visits before a fresh geolocation attempt resolves (research.md §8).

## Current Weather Snapshot

| Field | Type | Notes |
|---|---|---|
| `locationId` | string | FK → Location.id |
| `temperatureC` | number | Always stored in Celsius; converted for display via `units.js` (Principle II) |
| `windSpeedKmh` | number | Always stored in km/h; converted for display |
| `weatherCode` | integer | Open-Meteo WMO code → `{ icon, label }` via `weatherCode.js` |
| `isDay` | boolean | Drives day/night condition theme (research.md §12) |
| `todayLowC` / `todayHighC` | number | From the daily block for "today" |
| `observedAtLocalIso` | string | ISO timestamp in the **location's** local time (Edge Case: never the viewer's) |

## Hourly Forecast Entry

One per 3-hour step, 8 entries covering the next 24 hours (FR-006).

| Field | Type | Notes |
|---|---|---|
| `locationId` | string | FK → Location.id |
| `timeLocalIso` | string | Location-local timestamp of this slot |
| `temperatureC` | number | |
| `weatherCode` | integer | For the small per-point icon |

**Derivation rule**: Open-Meteo returns hourly data at 1-hour resolution;
`forecastBuckets.js` selects every 3rd hour starting from the current hour,
producing exactly 8 entries — this selection logic is one of the
Principle-IV-mandated unit-tested modules.

## Daily Forecast Entry

One per day, 5 entries (FR-007).

| Field | Type | Notes |
|---|---|---|
| `locationId` | string | FK → Location.id |
| `dateIso` | string | Calendar date (location-local) |
| `lowC` / `highC` | number | Drives the dumbbell chart's range bar (research.md §13) |
| `weatherCode` | integer | |

## Major City Summary

Lightweight record for the curated list (FR-008); a fixed, non-user-editable
set per spec.md Assumptions.

| Field | Type | Notes |
|---|---|---|
| `location` | Location | One of the ~5–8 curated cities, hardcoded in `src/data/majorCities.js` |
| `temperatureC` | number | |
| `weatherCode` | integer | |

**Relationship**: selecting a Major City Summary tile promotes that
`location` to the app-wide **Selected Location**, which then fetches the full
Current Weather Snapshot / Hourly / Daily sets for it (User Story 5,
Acceptance Scenario 2).

## Condition Theme (derived, not fetched)

Not a spec entity, but a computed value object every view reads from, so it's
recorded here for implementation clarity.

| Field | Type | Notes |
|---|---|---|
| `key` | string | e.g. `"clear-day"`, `"rain-night"` — `weatherCode` × `isDay` |
| `cssVars` | `Record<string, string>` | The `--sky-*` custom properties applied to the page root (research.md §12) |

Pure function: `(weatherCode, isDay) => ConditionTheme`, no network or state.
