# 📍 Pin & Plan

A trip-planning tool that lets you pin the places you want to visit directly on a map, attach your own research (notes, links, screenshots) to each pin, and get a suggested walking order automatically — instead of bouncing between a map app, a notes app, and a spreadsheet.

**Live demo:** _add your GitHub Pages link here once deployed, e.g. `https://yourusername.github.io/pin-and-plan/`_

## Why this exists

Planning a citywalk-style trip (café → boutique → restaurant → next stop) usually means researching spots on Xiaohongshu or a blog, saving them as scattered bookmarks or screenshots, then manually figuring out a sensible visiting order in a separate map app, then rebuilding all of it into an itinerary doc. Pin & Plan collapses that into one view: search or drop a pin, attach the research right on the pin, and let the app suggest the order.

## Features

- **Pin anything** — search by name (in any language) or click the map directly. Search results are scoped to your current city/region so "starbucks" doesn't return one from a different country.
- **Browse nearby by category** — Restaurants, Cafes, Shopping, Must-go, Bars, pulled live from OpenStreetMap.
- **Attach your own 攻略** — each pin holds free-text notes, a pasted link, and an image screenshot, so it becomes a self-contained, ready-to-use guide.
- **Automatic route optimization** — pins are automatically reordered into an efficient walking sequence (nearest-neighbor heuristic) with a real walking route drawn on the map and a distance/time estimate.
- **Multi-day, multi-trip** — organize pins into days with optional dates, and keep multiple separate trips.
- **One-tap navigation handoff** — jump straight into Google Maps, 高德地图 (Amap), or 百度地图 (Baidu Maps) for turn-by-turn directions.
- **Opening hours**, where available, shown on search results and pin cards.
- **Trip assistant** — a chat panel that answers questions about your plan locally (distance, timing, missing notes) by default, or connects to your own OpenAI/Anthropic API key for open-ended help.
- **Saves automatically** — everything persists in your browser between visits. No account, no login, nothing is lost when you close the tab.

## Tech stack

Single self-contained HTML file. No build step, no server, no framework.

- [Leaflet.js](https://leafletjs.com/) + OpenStreetMap tiles for the map
- [Nominatim](https://nominatim.org/) for place search (free, no API key)
- [Overpass API](https://overpass-api.de/) for category-based nearby search
- [OSRM](https://project-osrm.org/) for walking-route calculation
- Browser `localStorage` for persistence
- Optional: OpenAI or Anthropic API (bring your own key) for the AI trip assistant

## Running it locally

No install, no build. Download `index.html` and open it in any modern browser (Chrome, Safari, Edge). An internet connection is needed for map tiles, search, and routing.

## Deploying / updating

This is hosted as a static site (GitHub Pages). To update it after making changes, just replace `index.html` in the repo — GitHub Pages redeploys automatically within a minute or two. No versioning system or database to manage.

## Known limitations

- Data is stored per-browser (`localStorage`), not synced across devices — this is intentional for a single-user tool with no account system, but it does mean clearing browser data or switching devices/browsers loses your trips.
- Place search quality depends on OpenStreetMap/Nominatim coverage, which is generally weaker in mainland China than dedicated services like Amap or Baidu.
- The built-in trip assistant is a local rule-based heuristic unless you connect your own AI API key.

## Roadmap

See `PRD_pin_and_plan.md` for the fuller product requirements doc. Possible next steps: accounts + cross-device sync, shared/collaborative trips, native Amap/Baidu search integration for better China coverage.

## License

MIT — free to use, copy, and modify.
