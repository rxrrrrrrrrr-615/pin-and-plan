# Pin & Plan — portfolio write-up

Use this as a starting draft for your portfolio page. Trim or expand depending on how much space you have — the short pitch works as a card summary, the fuller sections work as a case-study page.

---

## Short pitch (for a portfolio card/thumbnail)

**Pin & Plan** — a map-first trip planner I designed and built end-to-end. Pin the places you want to visit, attach your own research to each stop, and get an automatically optimized walking route — solving the "app-switching" problem of citywalk-style trip planning.

[Live demo](#) · [GitHub](#)

---

## The problem

Every time I plan a citywalk-style trip (café → boutique → restaurant → next spot), I end up doing the same repetitive workflow: research places on Xiaohongshu or a blog, save them as scattered bookmarks or screenshots, open a separate map app to figure out a sensible visiting order, then rebuild everything into an itinerary doc so I actually have something to follow on the day. No existing tool let me do all three — pin, research, and route — in one place.

## The approach

Rather than jumping straight to code, I started by mapping the problem against existing tools (Wanderlog, Sygic Travel, Roadtrippers, Google My Maps, 马蜂窝) to understand what was already solved and where the real gap was. The finding: pin-and-route tools exist, but none let a pin carry its own personal research as a self-contained guide, and none work well in mainland China where Google Maps isn't usable.

I wrote a lightweight PRD to scope a v1: build for personal use first, validate the core loop (pin → attach research → get an optimized route → navigate), then decide whether it's worth extending toward a shared/public tool.

## What I built

A single-page web app where you can:

- Search or drop pins on a map (OpenStreetMap-based, works globally, not just in one country)
- Browse nearby places by category (restaurants, cafes, shops, attractions) pulled live from OpenStreetMap
- Attach notes, a link, and a screenshot to each pin as a ready-to-use guide
- Get pins automatically reordered into an efficient walking route (nearest-neighbor optimization, redrawn live as pins are added, moved, or removed)
- Organize pins across multiple days and multiple separate trips
- Jump straight into Google Maps, Amap, or Baidu Maps for actual turn-by-turn navigation
- Ask a built-in trip assistant questions about the plan (distance, timing, what's missing), with an optional bring-your-own-key connection to a real AI for open-ended help

## Design decisions worth calling out

- **No account system, no backend.** Since this started as a personal tool, everything saves locally in the browser. That kept it free to build and instant to deploy, while still meeting the actual requirement (don't lose my work when I close the tab).
- **Region-scoped search.** An early version of search returned identically-named places from other countries (a "Starbucks" in the wrong city). Fixed by anchoring search to the current map view instead of searching globally.
- **Automatic, not manual, route optimization.** Originally the "smart route" only ran when a button was clicked, so the map numbering didn't reflect an actual suggested order until you remembered to press it. Recalculating automatically whenever pins change made the numbering trustworthy by default.
- **Honest about what's real AI and what isn't.** The trip assistant is transparent about running on local heuristics unless the user connects their own API key, rather than pretending to be smarter than it is.

## What I'd do next

Documented in the full PRD: cross-device sync via a real account system, native Amap/Baidu search for better China-market place data, and testing with a small group of friends before considering any wider launch.

## Tech stack

Leaflet.js, OpenStreetMap (Nominatim + Overpass + tile layer), OSRM for routing, vanilla JavaScript, browser localStorage. No framework, no build step — deployed as a single static HTML file via GitHub Pages.
