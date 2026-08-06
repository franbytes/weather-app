# External API Contract (Consumed): Open-Meteo

This app has no backend of its own (Constitution Principle V), so its only
external interface is the third-party API it consumes. This contract pins
down exactly which endpoints, params, and response fields `src/api/openMeteo.js`
relies on, so implementation and tests build against a fixed shape rather than
the full upstream API surface.

Base URLs: `https://api.open-meteo.com` (forecast), `https://geocoding-api.open-meteo.com` (geocoding). No API key, no auth header.

## 1. Forecast — current + hourly + daily (single call)

`GET https://api.open-meteo.com/v1/forecast`

**Query params used**:

| Param | Value | Maps to |
|---|---|---|
| `latitude`, `longitude` | selected Location's coordinates | — |
| `timezone` | `"auto"` | Resolves times to the location's own IANA timezone (data-model.md "Current Weather Snapshot") |
| `current` | `temperature_2m,wind_speed_10m,weather_code,is_day` | Current Weather Snapshot |
| `hourly` | `temperature_2m,weather_code` | Hourly Forecast Entry (before 3-hour bucketing) |
| `daily` | `weather_code,temperature_2m_max,temperature_2m_min` | Daily Forecast Entry + today's high/low on the hero |
| `forecast_days` | `5` | Caps the daily payload to exactly what FR-007 needs |
| `temperature_unit` | always `celsius` | The app requests Celsius and converts for display itself (Principle II) — never asks the API for Fahrenheit |
| `wind_speed_unit` | always `kmh` | Same reasoning |

**Response shape relied upon** (fields consumed; extra upstream fields are
ignored):

```json
{
  "current": {
    "time": "2026-08-06T14:00",
    "temperature_2m": 21.4,
    "wind_speed_10m": 12.1,
    "weather_code": 2,
    "is_day": 1
  },
  "hourly": {
    "time": ["2026-08-06T00:00", "2026-08-06T01:00", "..."],
    "temperature_2m": [18.1, 17.9, "..."],
    "weather_code": [1, 1, "..."]
  },
  "daily": {
    "time": ["2026-08-06", "2026-08-07", "..."],
    "weather_code": [2, 61, "..."],
    "temperature_2m_max": [23.0, 19.5, "..."],
    "temperature_2m_min": [14.2, 13.1, "..."]
  }
}
```

`hourly.*` arrays are parallel and 1-hour resolution; `forecastBuckets.js`
zips `time`/`temperature_2m`/`weather_code` and keeps every 3rd index from the
current hour onward, taking 8. `daily.*` arrays are parallel, one entry per
day.

**Failure modes the client MUST handle** (Constitution Principle III):
non-2xx status, network error/timeout, and a malformed/missing `current` or
`daily` block → surface the shared error state, never a partial render.

## 2. Geocoding — city search

`GET https://geocoding-api.open-meteo.com/v1/search`

**Query params used**: `name` (the user's search text, debounced 300ms per
research.md §6), `count=5`, `language=en`, `format=json`.

**Response shape relied upon**:

```json
{
  "results": [
    {
      "id": 2643743,
      "name": "London",
      "admin1": "England",
      "country": "United Kingdom",
      "country_code": "GB",
      "latitude": 51.50853,
      "longitude": -0.12574,
      "timezone": "Europe/London"
    }
  ]
}
```

Missing `results` key (Open-Meteo omits it, rather than returning `[]`, when
there are zero matches) → treated as an empty array → renders the "No cities
found" empty state (FR-005).

## 3. Reverse geocoding — resolve a browser geolocation coordinate to a label

`GET https://geocoding-api.open-meteo.com/v1/reverse`

**Query params used**: `latitude`, `longitude` (from the browser Geolocation
API), `language=en`, `format=json`.

**Response shape relied upon**: same per-result shape as §2. Empty/failed
result → the app still has usable coordinates for the Forecast call, but
falls back to a generic `"Your location"` label instead of a city name.

## Client module contract (`src/api/openMeteo.js`)

The one module every hook/component is allowed to import for weather data
(Constitution Principle I):

```ts
getForecast(latitude: number, longitude: number): Promise<{
  current: CurrentWeatherSnapshot;
  hourly: HourlyForecastEntry[];   // raw 1-hour resolution; bucketing happens in forecastBuckets.js
  daily: DailyForecastEntry[];     // 5 entries
}>

searchCities(query: string): Promise<Location[]>       // §2, empty array on no matches
reverseGeocode(latitude: number, longitude: number): Promise<Location | null>  // §3
```

Each function throws a typed `WeatherApiError` on any failure mode listed
above; hooks catch it and populate the `{ data, error, isLoading }` shape
consumed by every view (research.md §4, Constitution Principle III).
