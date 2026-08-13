# Home Dashboard — Project Overview

## Summary

This repository contains a single-file, TV-friendly home dashboard implemented primarily in `index.html`. The dashboard is designed for 24/7 display (Raspberry Pi friendly) and combines weather, public-transport departures (HSL), electricity spot prices, calendars and rotating pages of content.

## Key files

- `index.html`: Single-file app with CSS, HTML and the entire JS runtime. Main entry and implementation.
- `quotes.txt`: Plain-text list of motivational quotes used on the Quotes page (separated by empty lines).
- `welcome.jpg`: Local welcome image used by the Welcome page (if present).
- `Misc/`: A separate folder containing other demo pages and a small Vite project (not required for the single-file dashboard).

## index.html — high level structure

- <head>
  - Large embedded `<style>` block that provides the dark glass aesthetic, responsive layout, and a Pi "lite" mode (class `lite`) which disables expensive effects.
- <body>
  - Visual overlays: `.bg-overlay`, `#condOverlay` for weather effects.
  - Top status bar: `header.status` with pills for clock, network, weather, HSL, electricity, calendar.
  - Persistent summary tiles: `.top-tiles` for compact, always-visible status (weather, next bus/train, electricity price).
  - Main pages: `main > .pages` contains several `.page` sections (page0..page5) including Today, Planning, School schedules, Quotes, Welcome.
  - Page navigation: `.page-dots` and router with automatic rotation.
  - Inline `<script>`: Implements configuration, state, cache, API fetchers, rendering, theme, router, scheduler, iframe handling and boot.

## Notable features (index.html)

- CONFIG object: global `CONFIG` at top of script controls rotation, refresh intervals, performance flags (Pi-friendly), timezone, location, HSL GraphQL endpoint and `apiKey`, electricity embed/API URLs, calendar/school embed URLs, and iframe pointer-event policy.
- Data sources:
  - Weather: Open-Meteo (`CONFIG.WEATHER.endpoint`).
  - HSL: Digitransit GraphQL (`CONFIG.HSL.graphqlUrl`) with an `apiKey` set in the file.
  - Electricity: `CONFIG.ELECTRICITY.apiBase` (JSON API) plus embeddable pages for visual charts.
  - Calendar & School: Google Calendar embed URLs in config.
  - Quotes: read from `quotes.txt` via fetch.
- Caching: `localStorage` keys for weather, HSL, electricity and quotes to enable offline/cached fallback.
- Backoff & retry: fetch logic uses exponential backoff and separate timers for each data source.
- UI rendering: `render` functions (statusBar, topTiles, weather, hsl, hslDetailed, quotes) throttle DOM updates by comparing computed render keys.
- Theme: `theme` computes time-of-day and weather-based color accents and optional condition overlays (clouds, rain, snow).
- Router & scheduler: automatic page rotation (`ROTATION_MS`) with manual navigation support and pause-on-interaction.
- Iframe handling: cache-busting for embed iframes, periodic refresh, and an option to disable pointer-events for iframes (for 24/7 displays).

## Where to customize

- Change location/timezone: edit `CONFIG.LOCATION` and `CONFIG.TIMEZONE`.
- Adjust performance: `CONFIG.PERF.LITE_MODE` and `CONFIG.PERF.REDUCE_MOTION`.
- Update HSL stops, API URL or API key: `CONFIG.HSL` (be aware the file currently contains an API key string).
- Replace calendar/school embed URLs in `CONFIG.CALENDAR` / `CONFIG.SCHOOL`.
- Edit `quotes.txt` to update Quotes page text (separate quotes with a blank line).

## Run / preview locally

Serving via simple HTTP is recommended (some embeds and fetches behave poorly over `file://`):

macOS / Python 3:

```bash
python3 -m http.server 8000
# then open http://localhost:8000 in a browser on the same machine
```

Or use VS Code Live Server or any static file server. For Raspberry Pi deployments, copy the folder to the Pi and serve with the same command or host via a lightweight web server.

## Security & privacy notes

- The HSL GraphQL API key is currently embedded in `index.html`. Rotate or remove it if you don't want it checked into source control.
- Iframes are embedded from third-party sites (electricity, Google Calendar). Some browsers or network policies may block these or require additional CORS/OAuth steps (calendar OAuth is noted in UI).

## Quick checklist / TODOs

- [ ] Replace placeholder school calendar URLs if you have per-child calendars.
- [ ] Review and (optionally) move the HSL `apiKey` to a separate config mechanism before committing publicly.
- [ ] Add `welcome.jpg` to the repo if you want a custom welcome image.
- [ ] If running on a low-powered Pi, keep `CONFIG.PERF.LITE_MODE = true` and `CONFIG.PERF.IFRAME_SCALE = 1.0`.

---

If you'd like, I can:

- extract the JavaScript into a separate `app.js` and the CSS into `styles.css` (to make the file easier to edit), or
- remove the embedded HSL API key and wire a `.env`-style loader, or
- run a quick local server and preview the dashboard in a headless browser.

Tell me which of these you'd like next.
