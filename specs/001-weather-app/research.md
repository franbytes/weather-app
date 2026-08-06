# Phase 0 Research: Weather App

**Input**: [spec.md](./spec.md) · **Constitution**: [.specify/memory/constitution.md](../../.specify/memory/constitution.md)

This document resolves every `NEEDS CLARIFICATION` left in `plan.md`'s Technical
Context, plus the design-direction decisions made using the `frontend-design`
and `dataviz` skills, as requested for this planning pass.

## 1. Weather data provider

**Decision**: [Open-Meteo](https://open-meteo.com) — Forecast API + Geocoding API.

**Rationale**:
- Free, no API key, no account, no request signing. This lets the app satisfy
  Constitution Principle V (Simplicity & Deployability) and the Technology &
  Delivery Constraints ("MUST run client-side with, at most, a minimal
  backend/proxy solely to protect a private API key **if** the chosen API
  requires one") by needing **zero backend or proxy at all** — the app can be a
  pure static build.
- One provider covers every data need in the spec: current conditions, hourly
  temperature (steppable to 3-hour intervals for FR-006), 5-day daily
  low/high + weather code (FR-007), and a Geocoding endpoint for city search
  that returns name + admin region + country (FR-005's disambiguation
  requirement) with lat/lon + time zone for FR-002/FR-003.
- Global coverage, generous free-tier rate limits appropriate for a portfolio
  demo's traffic.

**Alternatives considered**:
- **OpenWeatherMap** — industry-standard, well-known, but requires an API key.
  Exposing it client-side (unavoidable in a pure static SPA) means the key is
  effectively public; would force adding a serverless proxy purely for key
  hiding, which Principle V says to avoid unless the feature actually needs it.
- **WeatherAPI.com** — also key-gated, same tradeoff as above; no material
  capability advantage for this spec's requirements.

## 2. Front-end stack

**Decision**: React 18 + Vite (already resolved in `/speckit-clarify`; restated
here for completeness). Vite gives near-zero-config dev/build, aligning with
Principle V.

## 3. State management

**Decision**: React Context + hooks only (`SelectedLocationContext`,
`UnitPreferenceContext`); no external state library (Redux, Zustand, etc.).

**Rationale**: The app's shared state is small — one selected location, one
unit preference, and per-view server-cache data. Constitution Principle V
explicitly says to avoid a state-management library unless the feature set
requires one; it doesn't here.

**Alternatives considered**: Redux Toolkit / Zustand — rejected as
unnecessary ceremony for two small pieces of global state.

## 4. Data fetching & caching

**Decision**: A small custom hook layer (`useCurrentWeather`, `useForecast`,
`useCitySearch`) built on the native `fetch` API, each returning
`{ data, error, isLoading }`. A lightweight in-memory cache keyed by
`lat,lon` avoids redundant calls when the user revisits a location in the same
session.

**Alternatives considered**: React Query / SWR — capable libraries, but add a
dependency and a learning-curve for caching behavior this app doesn't need at
its scale (a handful of requests per session). Rejected under Principle V;
revisit only if manual cache handling proves error-prone during implementation.

## 5. Geolocation & default location

**Decision**: On first load, request the browser Geolocation API. If granted,
reverse-resolve coordinates to a location label via Open-Meteo's reverse
geocoding; if denied, unavailable, or it times out (5s), fall back to a fixed
default city (**London, GB**). This directly implements FR-003 and the
"location detection denied" edge case.

## 6. City search & disambiguation

**Decision**: Debounced (300ms) query against Open-Meteo Geocoding as the user
types; render up to 5 results, each labeled `"{name}, {admin1}, {country}"` so
same-named cities are distinguishable (FR-005). No results → an explicit
"No cities found for '{query}'" empty state (Constitution Principle III).

## 7. Unit conversion

**Decision**: One pure module, `src/utils/units.js`, exporting
`toDisplayTemp(celsiusValue, unit)` and `toDisplayWind(kmhValue, unit)`. Every
component reads temperature/wind through this module — never converts inline.
This is the direct implementation of Constitution Principle II
(Consistent Units Everywhere, NON-NEGOTIABLE).

## 8. Persistence

**Decision**: `localStorage` for the last-selected location and unit
preference (per spec.md Assumptions), read once on load and written on
change. No cookies, no backend session — consistent with "no accounts" scope.

## 9. Testing

**Decision**: **Vitest** (native Vite integration, Jest-compatible API) +
**React Testing Library** for component/interaction tests. Satisfies
Constitution Principle IV: unit tests are required for `units.js` (conversion
correctness), the 3-hour bucketing selector, the 5-day rollup mapper, and the
city-search disambiguation/label logic before those pieces are considered done.

## 10. Deployment target

**Decision**: **Vercel**, deploying the Vite static build output. Zero-config
for a Vite SPA, generates a public demo URL on every push, satisfies FR-012 /
SC-006 and the constitution's single-build-step requirement.

**Alternatives considered**: Netlify, GitHub Pages — equally valid static
hosts; Vercel chosen for the fastest zero-config path from a GitHub repo to a
public URL, which is all this project needs.

## 11. Weather icons

**Decision**: A small (~10 glyph) hand-authored SVG icon set — clear-day,
clear-night, partly-cloudy(-night), cloudy, fog, drizzle, rain, snow,
thunderstorm — mapped from Open-Meteo's WMO weather codes, drawn to match the
"Almanac Console" aesthetic (§12) rather than pulling in a generic icon
library. Every status is always paired with its text label (e.g., "Partly
Cloudy"), never conveyed by icon/color alone.

**Alternatives considered**: A general icon package (e.g., a full weather-icon
npm library) — rejected: pulls far more glyphs than the ~10 WMO groupings this
app needs and would fight the custom aesthetic direction below.

---

## 12. Design direction (via `frontend-design` skill)

**Aesthetic**: **"Almanac Console"** — a warm, editorial almanac/weather-bulletin
voice for headline information, layered with a precise instrument-panel
("telemetry") voice for dense data. Deliberately avoids the generic
glassmorphism-on-purple-gradient look common to AI-generated weather-app
mockups.

- **Typography**: Display face **Fraunces** (variable, soft-serif with real
  character) for the hero temperature, city name, and section headers — reads
  like a vintage almanac bulletin. Data face **Fragment Mono** (fallback: IBM
  Plex Mono) for every numeric readout that isn't the hero figure: timestamps,
  wind speed, coordinates-adjacent labels, forecast strip values, and chart
  tick labels — reinforces the "instrument reading" feel and gives aligned
  columns via `font-variant-numeric: tabular-nums`.
- **Color / theming**: A `--sky-*` CSS custom-property group is swapped per
  **weather condition × day/night**, producing an abstract gradient-mesh
  background (not photographic) with a subtle grain overlay — e.g., clear-day
  is warm gold-to-sky-blue, clear-night is deep indigo-violet with a fine star
  texture, rain is cool slate-blue with a faint diagonal-line texture, snow is
  pale cyan-white, storm is charcoal with a sharp amber accent flash. This
  theme layer governs page chrome only (background, accent buttons, active
  states) — it is intentionally kept separate from the data-visualization
  color layer (§13) so temperature charts stay legible and consistent no
  matter which condition theme is active.
- **Layout**: An asymmetric bento grid rather than a stack of equal cards: the
  current-weather hero panel bleeds to the container edge and takes visual
  priority; the 24-hour strip is a horizontal-scroll instrument strip beneath
  it; the 5-day forecast is a compact vertical list beside/below it; the
  major-cities summary is a dense mosaic of small "instrument tiles." Generous
  negative space around the hero, tighter rhythm in the data-dense areas.
- **Motion**: One staggered reveal on load (hero, then hourly strip items,
  then daily list, then city tiles — via CSS `animation-delay`), plus a
  hover/press state on interactive tiles. Pure CSS animations/transitions,
  refined in §14 for the two genuinely gesture-driven interactions.
- **Unit toggle**: Styled as a physical two-position switch (fits the
  instrument-panel motif) rather than a generic pill/checkbox; its motion
  treatment is decided in §14.

## 13. Data visualization approach (via `dataviz` skill)

Two charts are needed; both were chosen by **job**, per `choosing-a-form.md`,
before any color decision:

- **24-hour forecast (8 points)** — job: *trend over time, single series* →
  a thin-line **area/sparkline** chart, custom inline SVG (`<path>` + a
  low-opacity gradient fill), not a charting library. At 8 points, a library
  (Recharts/Victory/etc.) would be pure overhead against Principle V; the
  mark spec (2px line, 4px rounded data-ends, ≥8px hit targets) is simple
  enough to hand-roll. Every point is direct-labeled with its temperature and
  weather icon (small-n dataset — direct-labeling all 8 is appropriate per
  the marks guidance), with a tap/hover crosshair for the exact reading.
- **5-day forecast (low/high per day)** — job: *before → after per item* →
  a **dumbbell** (floating range bar) per day, one shared temperature axis
  across all 5 days so the days are visually comparable (never independently
  rescaled — the "one axis" non-negotiable).
- **Color**: Both charts are single-hue **sequential** encodings of
  temperature magnitude (not identity), so no categorical/CVD-adjacency
  validation is required — the skill's category ladder only forces that check
  once a form needs to distinguish 2+ *identities* by hue. Position on the
  chart already carries the primary signal; color reinforces it. Both charts
  reuse the `dataviz` skill's **validated reference sequential ramp** (blue,
  steps 250–650: `#86b6ef → #5598e7 → #2a78d6 → #1c5cab`) unchanged — coolest
  reading maps to the lightest step, warmest to the darkest. Because this is
  the published reference instance, its only sequential requirement
  (lightness monotonicity) is already satisfied by construction; no
  categorical-adjacency script run is needed for a single-hue ramp.
- **Chart surface**: The two charts sit on the skill's neutral reference card
  surfaces (light `#fcfcfb` / dark `#1a1a19`), deliberately distinct from the
  atmospheric "Almanac Console" page background (§12). This keeps the
  temperature ramp's color meaning constant and legible regardless of which
  weather-condition theme is currently active on the page.
- **Accessibility**: A text/table equivalent (a plain list of the same 8
  hourly and 5 daily values) is available alongside each chart; weather
  status is always icon + label together, never color-only (consistent with
  the skill's status-color rule).

**Alternatives considered**: A full charting library (Recharts, Chart.js,
Victory) — rejected per Principle V for datasets this small (8 and 5 points);
a bar chart for the hourly view — rejected, "trend over time" reads better as
a line/area than discrete bars at this cadence.

## 14. Motion & materials (via `apple-design` skill)

Applied selectively — only where an interaction is genuinely gesture-driven or
where translucency communicates layering. Everywhere else stays plain CSS,
respecting Principle V.

- **Interruptible spring motion, scoped to two components**: the
  **unit toggle** (°C/°F switch) and the **city-search results panel**
  (open/select/dismiss) use a small (~2.5kB) spring engine
  (`motion` / motion.dev's core, not the full React wrapper) instead of a CSS
  transition, because both are things a user "flips" or "picks" and expects to
  feel physically responsive and re-grabbable mid-motion:
  - Unit toggle thumb: critically damped spring (`damping 1.0`,
    `response ≈ 0.3s`) driven by the boolean state, not a `left`/`transform`
    CSS transition — pressing it mid-flight re-targets smoothly instead of
    snapping.
  - Search panel open/close: same critically-damped default; no bounce, since
    neither gesture carries flick momentum.
  - This is the **one deliberate, scoped exception** to "no extra front-end
    dependency": a 2.5kB single-purpose engine used in exactly two components,
    not a general animation framework. Everything else (page-load stagger,
    hover/press states, theme cross-fades) remains pure CSS.
  - Every interactive control (toggle, search result row, city tile) gets its
    pressed-state feedback via `:active` on **pointer-down**, not on click —
    consistent with "kill latency" (§1 of the skill).
- **Translucent "instrument glass" panels**: the search-results dropdown and
  the hover/focus elevation on a city tile use a frosted-glass material
  (`backdrop-filter: blur(20px) saturate(180%)` over a semi-transparent
  surface) rather than an opaque card, so they read as a pane of glass over
  the live gradient "sky" background — reinforcing the instrument-panel
  motif. Text on these surfaces uses higher contrast and a touch more weight
  than the flat-background default (vibrancy), and color always sits on a
  solid layer beneath, never on the translucent foreground itself. Two glass
  layers are never stacked directly on each other.
- **Hourly-forecast strip scrolling**: relies on native horizontal touch/track
  scrolling with CSS `scroll-snap-type: x proximity` for the rubber-band and
  momentum feel already built into the browser/OS — no custom drag-physics
  code, keeping Principle V intact.
- **Reduced motion & transparency, required, not optional**: every animated
  or glass surface has a fallback:
  - `prefers-reduced-motion: reduce` → the toggle and search panel drop the
    spring in favor of a short opacity cross-fade; the load-in stagger
    collapses to a simple fade.
  - `prefers-reduced-transparency: reduce` → glass surfaces raise background
    opacity and drop the blur, becoming near-solid.
  - `prefers-contrast: more` → glass/gradient surfaces fall back to a solid
    surface color with a defined border.
- **Typography detail**: the Fraunces hero temperature figure uses
  size-specific negative tracking (`letter-spacing: -0.02em` at its largest
  size) and tight leading (`line-height: 1.05`); body and Fragment Mono data
  text stay near `letter-spacing: 0` with more relaxed leading, per the
  skill's size-specific tracking/leading rule.

**Alternatives considered**: Framer Motion's full React API — rejected as
heavier than needed when only two components require spring physics; CSS
`transition`/`@keyframes` for the toggle and search panel — rejected per the
skill's core finding that they cannot be smoothly re-grabbed/reversed
mid-motion, which matters for a control the user taps repeatedly.
