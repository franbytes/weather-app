# Weather App

A weather app that shows current conditions, an hourly forecast, a 5-day
outlook, and at-a-glance conditions for major world cities — with a search-any-city
flow and a Celsius/Fahrenheit toggle.

**Live demo**: _coming soon_
**Status**: In development — see [specs/001-weather-app/](specs/001-weather-app/) for the full spec-driven design docs.

## Features

- Current weather for a location: temperature, name, local time, wind, today's high/low, and condition
- 24-hour forecast in 3-hour intervals (8 points)
- 5-day forecast with condition and low/high per day
- At-a-glance summary for a curated set of major world cities
- City search with disambiguation (e.g. same-named cities in different countries)
- Celsius / Fahrenheit toggle, applied consistently everywhere

## Tech Stack

- **React 18 + Vite** — front end, static build, no backend
- **[Open-Meteo](https://open-meteo.com)** — weather, forecast, and geocoding data (free, no API key)
- **Vitest + React Testing Library** — tests
- Deployed as a static site (Vercel)

See [specs/001-weather-app/research.md](specs/001-weather-app/research.md) for
the full rationale behind these choices, including the design direction
("Almanac Console") and data-visualization approach.

## Getting Started

```bash
npm install
npm run dev       # starts the dev server
npm run test      # runs the test suite
npm run build     # production build to dist/
```

## Project Documentation

This project was built with [GitHub Spec Kit](https://github.com/github/spec-kit)
following a spec-driven workflow. All design artifacts live under
[`specs/001-weather-app/`](specs/001-weather-app/):

| Doc | Purpose |
|---|---|
| [spec.md](specs/001-weather-app/spec.md) | Feature specification: user stories, requirements, success criteria |
| [plan.md](specs/001-weather-app/plan.md) | Implementation plan: tech stack, architecture, constitution compliance |
| [research.md](specs/001-weather-app/research.md) | Technical & design decisions with rationale |
| [data-model.md](specs/001-weather-app/data-model.md) | Data entities and shapes |
| [contracts/](specs/001-weather-app/contracts/) | External API contract (Open-Meteo) |
| [tasks.md](specs/001-weather-app/tasks.md) | Actionable implementation task list |

Project governance and non-negotiable principles are in
[.specify/memory/constitution.md](.specify/memory/constitution.md).

## Branching

- `main` — stable/integration branch
- `001-weather-app` — active feature branch for this app's initial build (spec → plan → tasks → implementation), merged to `main` via pull request once complete
