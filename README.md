# 📍 Pin & Plan

A trip-planning tool that lets you pin the places you want to visit directly on a map, attach your own research (notes, links, screenshots) to each pin, and get a suggested walking order automatically — instead of bouncing between a map app, a notes app, and a spreadsheet.

**Live demo:** _add your GitHub Pages link here once deployed, e.g. `https://yourusername.github.io/pin-and-plan/`_

**Version:** v0.1.1 (stability-fix round; see `PRD_pin_and_plan.md` for the version history)

## Why this exists

Planning a citywalk-style trip (café → boutique → restaurant → next stop) usually means researching spots on Xiaohongshu or a blog, saving them as scattered bookmarks or screenshots, then manually figuring out a sensible visiting order in a separate map app, then rebuilding all of it into an itinerary doc. Pin & Plan collapses that into one view: search or drop a pin, attach the research right on the pin, and let the app suggest the order.

## Features

- **Pin anything** — search by name (in any language), press Enter or tap 🔍 to run the search (it does not fire on every keystroke), or click the map directly to drop a pin. Search results are scoped to your current city/region so "starbucks" doesn't return one from a different country.
- **Browse nearby by category** — Restaurants, Cafes, Shopping, Attractions, Bars, pulled live from OpenStreetMap (Overpass API).
- **Persistent per-pin categories** — each pin can carry one of the same five categories, shown as a badge on its card and editable any time.
- **Attach your own 攻略** — each pin holds free-text notes, a pasted link, and an image screenshot (automatically downscaled to a 1280px long edge at JPEG quality 0.7 before it's saved, to stay within the browser's storage limit), so it becomes a self-contained, ready-to-use guide.
- **AI-assisted parsing ("Parse with AI")** — with your own OpenAI or Anthropic key connected (this turns on **AI Mode**), ask the assistant to suggest a cleaned-up name and summary from whatever's attached to a pin: your notes, the screenshot, and the link's raw URL text. The AI never fetches or visits the link — it only ever sees it as plain text, and is explicitly instructed not to guess a place name from a bare URL alone. A hint under the link field says this directly.
- **Automatic route ordering** — pins are automatically reordered into a suggested walking sequence (nearest-neighbor heuristic — a suggestion, not a guaranteed-optimal or AI-generated route), recalculated live as pins are added, moved, or removed. Manual drag-and-drop or the ▲/▼ "earlier/later" buttons let you override the order directly; doing so pauses auto-reordering for that trip until "Recalculate route" is pressed.
- **Honest route distance/time** — when the walking-route service is reachable, a real routed line is drawn and its actual distance/duration is shown, labeled "route". When it isn't (or times out after 8 seconds), a dashed straight-line estimate is shown instead and labeled "approx" — never presented as a measured route.
- **Multi-day, multi-trip** — organize pins into days with optional dates, and keep multiple separate trips. (There is currently no way to delete a trip once created — see Known limitations.)
- **One-tap navigation handoff** — jump straight into Google Maps, 高德地图 (Amap), 百度地图 (Baidu Maps), or Apple Maps for turn-by-turn directions, with coordinates converted to each service's own coordinate system so pins land in the right place.
- **Opening hours**, where available, shown on search results and pin cards.
- **Trip assistant** — a chat panel with two modes, always shown in the header: **Local Mode** (default, no key — answers questions about distance, timing, and missing notes using simple heuristics, not a real language model) or **AI Mode** (connect your own OpenAI/Anthropic API key for open-ended help and the AI parsing feature above).
- **Sample trip** — a "Sample trip 示例行程" button loads a new trip with 6 real, Nominatim-verified Toronto Old Town landmarks and neutral factual notes, so the app is useful to try even if search is unreachable. It never loads automatically and never touches your existing trips.
- **No native browser dialogs** — every `alert()`/`confirm()`/`prompt()` has been replaced with an in-page toast (passive notices) or modal (anything needing a decision), since native dialogs can be blocked or behave inconsistently inside embedded/in-app browsers.
- **Saves automatically** — trip and pin data persists in your browser (`localStorage`) between visits; no account or login needed. If a save ever fails because you're at the browser's storage limit, you get a clear toast instead of silently losing the change. Your AI API key, if you add one, is kept only in that browser tab's memory (`sessionStorage`) and clears when you close the tab.
- **Zero third-party page dependencies** — Leaflet's JS and CSS are inlined directly into `index.html` and Google Fonts has been removed in favor of a system font stack (including PingFang SC and Microsoft YaHei for Chinese text), so the page makes no third-party script/stylesheet requests. It still makes live network calls to OpenStreetMap tiles, Nominatim, Overpass, and the routing service, since those are the live data the app needs.

## Tech stack

Single self-contained HTML file. No build step, no server, no framework, no third-party script/stylesheet dependencies (Leaflet is vendored inline).

- [Leaflet 1.9.4](https://leafletjs.com/) (inlined) + OpenStreetMap tiles for the map
- [Nominatim](https://nominatim.org/) for place search (free, no API key), explicit-trigger only (Enter/button), not fired on every keystroke
- [Overpass API](https://overpass-api.de/) for category-based nearby search
- `routing.openstreetmap.de/routed-foot/` (an OSRM instance that actually serves a pedestrian profile — see Known limitations) for walking-route calculation, with a labeled straight-line fallback when it's unreachable or times out (8s)
- Browser `localStorage` for trip/pin persistence; browser `sessionStorage` (cleared on tab close) for the optional AI API key
- Optional: OpenAI (`gpt-4o-mini`) or Anthropic (`claude-haiku-4-5-20251001`, vision-capable) API — bring your own key, never stored on any server — for AI Mode (trip assistant chat + AI-assisted pin parsing, including screenshots)
- System font stack (`-apple-system`, `PingFang SC`, `Microsoft YaHei`, `Segoe UI`, `Roboto`, `Arial`, etc.) — no Google Fonts or other web-font requests

## Running it locally

No install, no build. Download `index.html` and open it in any modern browser (Chrome, Safari, Edge). An internet connection is needed for map tiles, search, nearby-browse, and routing — the page itself has no other external dependencies.

## Deploying / updating

This is hosted as a static site (GitHub Pages). To update it after making changes, just replace `index.html` in the repo — GitHub Pages redeploys automatically within a minute or two. No versioning system or database to manage.

## Known limitations

- Data is stored per-browser (`localStorage`), not synced across devices — this is intentional for a single-user tool with no account system, but it does mean clearing browser data or switching devices/browsers loses your trips. There is also no way to delete a trip once created (rename or repurpose it instead); this is a known, pre-existing gap.
- `localStorage` has a small per-site cap (typically a few MB). Screenshots are downscaled before saving to reduce how fast you hit it, and a save that still fails shows a clear toast rather than losing the change silently — but the cap itself hasn't changed.
- Place search quality depends on OpenStreetMap/Nominatim coverage, which is generally weaker in mainland China than dedicated services like Amap or Baidu. Reachability of the app's services (map tiles, Nominatim, Overpass, the routing service, Google Fonts previously) from inside mainland China has not been independently tested as of this version.
- The walking-route service is a free, best-effort public OSRM demo instance (`routing.openstreetmap.de/routed-foot/`) with no uptime guarantee. A closely related but different public demo server, `router.project-osrm.org`, was tested live and found to silently return car-driving routes when asked for the `/foot/` profile (confirmed on three independent test routes) — this app does not use that endpoint for walking routes.
- The Local Mode trip assistant is a rule-based heuristic, not a real language model, unless you connect your own AI API key to switch on AI Mode.
- AI-assisted parsing never fetches the content behind a pasted link — it only ever sees it as raw text, and is instructed not to guess a place name from a bare URL alone. For links whose content matters, paste the relevant details into notes or attach a screenshot instead.
- Your AI API key lives only in `sessionStorage` for the current browser tab — it must be re-entered after closing the tab, by design.
- `gpt-4o-mini` availability was confirmed via secondary sources, not OpenAI's own docs page directly, as of this version — worth periodically re-checking.

## Roadmap

See `PRD_pin_and_plan.md` for the fuller product requirements doc, including the confirmed next priorities (user-controlled route constraints — a daily time budget plus a "locked" flag that protects a pin from over-budget removal suggestions — and collaborative planning) and a lightweight UI/UX polish pass, both scoped but not yet built as of v0.1.1.

## License

MIT — free to use, copy, and modify.
