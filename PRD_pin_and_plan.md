# Pin & Plan — Product Requirements Document

**Status:** Draft v1 (feature/strategy sections) · implementation sections updated for **v0.1.1** · **Owner:** Rebecca · **Stage:** 0-to-1, pre-launch prototype in personal use

---

## Problem Statement

Travel planning today is scattered across too many disconnected tools. People discover places to go through 小红书, blogs, or Google, save them as scattered bookmarks or screenshots, then manually rebuild that list into a PPT, itinerary doc, or spreadsheet. Separately, they open a map app to figure out which order to visit things in, then switch to yet another app to actually book tickets or navigate. This is especially painful for "citywalk" style trips, common on girls' trips, where the plan is a loose sequence of small stops (a café, then a boutique, then a photo spot, then a restaurant) rather than a few big-ticket attractions. There is no single place to both decide *where* to go and *in what order*, and no existing tool lets a pinned place carry its own personal research (a pasted link, a screenshot, a note) as a ready-to-use guide. The cost of not solving this is hours of manual, repetitive replanning for every trip, and a plan that lives in three or four different apps at once instead of one.

This is grounded in Rebecca's own recurring pain point across multiple trips, not yet validated with a broader user sample — validating it with friends is part of this plan (see Phasing).

## Goals

1. Cut the time to go from "list of places I want to visit" to "an ordered, ready-to-use plan" to under 15 minutes for a single-day citywalk.
2. Every pinned place carries its own attached research (link, note, image, and now an optional AI-assisted parse of that research) so the plan is self-contained — no bouncing back to 小红书/blogs to remember why a place was saved.
3. Works the same way regardless of country for navigation handoff — a trip in Toronto and a trip in Chengdu use the same tool, with one-tap handoff to whichever map app (Google Maps, Amap, Baidu Maps, or Apple Maps) is actually usable in that country. Place *search* quality still depends on OpenStreetMap/Nominatim coverage, which is weaker in mainland China than dedicated services (see Known Limitations in README.md).
4. Rebecca and her friend group actively use it as their default trip-planning tool for at least 2 real trips before considering any public launch.
5. (Business goal, later phase) Validate whether this is worth building into a public product at all — this PRD explicitly treats that as unproven until Goal 4 is met.

## Non-Goals

- **Booking flights, hotels, or tickets in-app.** Redirecting to a map app for navigation is in scope; becoming an OTA is not. Too complex, and duplicates well-solved tools.
- **Building a native mobile app in v1.** The browser-based prototype is the vehicle for validating the idea. A native app is a possible future phase, not a v1 concern.
- **Full social/discovery features** (following other users, browsing public trip feeds, recommendations from strangers). This is a planning tool for a small trusted group first, not a travel content platform. Revisit only if Goal 4 is met and there's a real signal people want to discover, not just plan.
- **Real-time multi-user collaborative editing** (Google-Docs-style simultaneous editing) in v1. Async sharing (send a link, friend edits, changes sync on reload) is enough for the friend-group phase.
- **Monetization.** Explicitly out of scope for now per current direction — free, no business model yet. Revisit only after real usage validates the product is worth building further.

## User Stories

**As a trip organizer planning a citywalk with friends,**
- I want to search for or drop a pin on any place I'm considering, so that I can build my "maybe" list visually on a map instead of in a separate note.
- I want to attach a link, a pasted note, or a screenshot to each pin, so that the pin itself carries all the context I researched, and I don't have to re-find that 小红书 post later.
- I want AI to help turn what I attached (notes, a screenshot, or a link) into a clean place name and summary, so I don't have to manually rewrite my research into something usable. (Implemented as opt-in "Parse with AI", gated behind my own API key; it does not fetch the link itself — only notes and the screenshot are actually read.)
- I want the tool to suggest an efficient walking order for my pinned stops, so that I don't have to manually reason about which store is closer to which restaurant, and I want to be able to override that order by hand (drag, or move earlier/later) when I know better, without the tool silently re-sorting my manual choices.
- I want to see a rough walking distance and time for my planned route, so that I know if I'm overplanning a single day — and I want to know whether that number is a real measured route or just an estimate.
- I want to tap a pin and open it directly in Google Maps, Amap, Baidu Maps, or Apple Maps, so that I can actually get walking directions using whichever app works in the country I'm in.
- I want my pins and notes to persist when I close and reopen the tool, so that I don't lose my planning work.

**As a friend invited to help plan or follow the trip,**
- I want to view (and ideally edit) the same trip plan the organizer built, so that we're planning together instead of over text messages. *(Not yet built — see P1.)*
- I want to see whose 攻略/notes are attached to which pin, so that I know the source of a recommendation, especially if multiple friends contributed spots. *(Not yet built — see P1.)*

**As either user, on the day of the trip,**
- I want to open the trip plan on my phone and quickly get walking directions to the next stop, so that I'm not fumbling between apps mid-walk.
- I want to reorder or remove a pin on the fly if plans change (a shop is closed, we're tired, we found something better), so that the plan flexes with the actual day rather than being a fixed document.

**Edge cases**
- As a user, if I search for a place with no results or a typo, I want a clear "no results" state rather than a silent failure.
- As a user, if the route-optimization service is temporarily unavailable, I still want to see a straight-line route and distance estimate rather than nothing, and I want it to be obvious that it's an estimate and not a real route.
- As a user, if I have zero or one pin, I want the tool to not attempt route optimization and instead prompt me to add more pins.
- As a user, if a save to browser storage fails (e.g. storage is full), I want a clear notice that my latest change wasn't saved, not a silent loss of data.

## Requirements

### Must-Have (P0) — built and working as of v0.1.1
- [x] Drop a pin by clicking the map or searching a place name (free geocoding via Nominatim, works in any country), search triggered explicitly by Enter or a search button — not on every keystroke
- [x] Browse nearby places by category (Restaurants, Cafes, Shopping, Attractions, Bars) via Overpass, and assign a persistent category to any pin
- [x] Attach free-text notes, a pasted link, and a screenshot to each pin; screenshots are downscaled client-side before saving (1280px long edge, JPEG q0.7) to stay within browser storage limits
- [x] AI-assisted parsing ("Parse with AI") of a pin's attached notes/screenshot (and the link, as raw text only — never fetched) into a suggested name + summary, gated behind a connected OpenAI or Anthropic key (bring-your-own-key, key kept only in `sessionStorage` for the current tab, never stored server-side)
- [x] Auto-generated suggested visiting order (nearest-neighbor heuristic — explicitly not called "optimal" or "AI" anywhere in the UI), recalculated live as pins change, with manual override (drag-and-drop or ▲/▼ buttons) that pauses auto-reordering until "Recalculate route" is pressed
- [x] Real walking-route line with distance/time when the routing service is reachable (clearly labeled "route"), with a straight-line fallback (clearly labeled "approx") if it's down, returns no route, or times out after 8 seconds — never presented as if it were a measured route
- [x] One-tap redirect from any pin to Google Maps, Amap, Baidu Maps, or Apple Maps, with coordinates converted to each service's own coordinate system
- [x] Opening hours shown where available from the underlying map data
- [x] Data persists locally between sessions (`localStorage`), with a clear toast (not a silent failure) if a save ever fails because storage is full
- [x] Works the same in any country for navigation handoff — no hardcoded region or default map location; either resumes the last view or starts from the current location
- [x] No native browser dialogs (`alert`/`confirm`/`prompt`) — all replaced with in-page toasts/modals, since native dialogs can be blocked or behave inconsistently in embedded/in-app browsers
- [x] Zero third-party script/stylesheet dependencies (Leaflet vendored inline; no Google Fonts) — map tiles, search, nearby-browse, and routing remain live network calls, which is expected

**Acceptance criteria (example, for route ordering):**
- Given at least 2 pins exist on the map
- When pins are added, moved, or removed (and the trip is not in manual-order mode)
- Then the pin list re-orders automatically into a suggested walking sequence, a route line is drawn on the map, and the estimated distance/time updates and is labeled as either a real route or an estimate
- Given the routing service is unreachable, returns no route, or doesn't respond within 8 seconds
- When a route is requested
- Then a straight-line route and rough distance estimate are still shown, labeled "approx", and no error is surfaced to the user
- Given the user has manually dragged or reordered pins
- Then that order is kept as-is (not silently re-sorted) until "Recalculate route" is pressed

### Nice-to-Have (P1) — confirmed next priorities
- [ ] **User-controlled route constraints:** a daily time budget (walking + a per-pin "stay" duration), and a "locked" flag on a pin so the tool can suggest — never silently make — which unlocked pin to move if the day is over budget. Route *ordering* logic itself does not change; lock only affects removal suggestions. Scoped as the next major feature; not yet built as of v0.1.1.
- [ ] **Collaborative planning:** a friend can contribute places, see whose notes are whose, and edit the plan without overwriting someone else's change. Requires moving off pure per-browser `localStorage` to accounts + a shared backend — a real architecture shift, not yet started.
- [ ] Lightweight UI/UX polish pass (calmer visual hierarchy, a single clearer navigation-menu button instead of 4 separate map buttons, mobile-width fixes) — scoped, not yet built.
- [ ] Trip deletion — there is currently no way to remove a trip once created (only rename or clear its pins); flagged as a known gap.
- [ ] Mobile-friendly layout pass beyond what already works (current prototype is usable but not yet audited at 375–430px widths)

### Future Considerations (P2) — explicitly deferred, but should not be architecturally blocked
- [ ] A managed/hosted AI assistant that doesn't require the user's own API key — the current AI features (chat + parse) work today only when the user supplies their own OpenAI/Anthropic key
- [ ] Native mobile app for offline use and better on-the-go navigation handoff
- [ ] Native Amap/Baidu search integration (better China place-search quality than free OSM/Nominatim data)
- [ ] Opening-hours and crowd-time awareness when suggesting route order
- [ ] Public trip templates or discovery (explicitly non-goal for now, but don't design the data model in a way that makes this impossible later)
- [ ] Cross-device account sync independent of collaborative editing (so a solo user doesn't lose data by switching devices/browsers)

## Implementation Notes — v0.1.1 (stability-fix round)

This section documents technical facts about the current build, verified directly (live network tests, current vendor docs, or direct code/repo inspection) rather than assumed. It does not report on user testing, since none has happened in this round.

- **Search is explicit-trigger only.** Both the main place search and the city picker fire on Enter or a search button click — never on every keystroke — with request cancellation so a repeat search can't pile up stale responses.
- **Amap and Baidu coordinate conversion fixed.** Both services use their own encrypted coordinate systems (GCJ-02 / BD-09), not raw GPS/WGS84. The navigation links now correctly tell each service "this input is WGS84, please convert it" (`coordinate=wgs84` for Amap, `coord_type=wgs84ll` for Baidu) — previously this was missing or wrong, which silently offset markers.
- **Walking-route honesty and endpoint.** Live-tested three independent routes (Shanghai, Chengdu, Toronto): `router.project-osrm.org`'s `/foot/` endpoint silently returns car-driving geometry on this particular public demo server (confirmed identical output to its own `/driving/` profile, and speeds of 24–38 km/h). Switched to `routing.openstreetmap.de/routed-foot/`, a separate FOSSGIS-operated OSRM demo instance, confirmed via the same three test routes to return genuine pedestrian routing (~4.5 km/h implied speed, correct `mode:"walking"`). The displayed distance/time is now always labeled as either a real route or a straight-line estimate, matching whichever was actually used.
- **Route drawing robustness.** A request-sequence counter prevents a slow, superseded network response from overwriting a newer one; an 8-second timeout guarantees the app never hangs waiting on the routing service; the map's auto-fit-to-bounds behavior is now scoped to trip/day switches and "Recalculate route" only, instead of firing on every pin add or drag.
- **AI key storage.** Moved from `localStorage` (persists indefinitely) to `sessionStorage` (cleared when the tab closes).
- **AI model.** `claude-haiku-4-5-20251001` — confirmed against Anthropic's current model docs to support image input, so one model now covers both chat and screenshot parsing (previously the code switched to a different model for images). Error messages now name the provider and include the underlying HTTP status rather than a generic "check your key" message. `gpt-4o-mini` (OpenAI side) is kept, based on secondary-source confirmation that it remains available as of this version — not confirmed against OpenAI's own docs page directly, so worth re-checking periodically.
- **Parse-with-AI honesty.** The prompt sent to the model explicitly forbids guessing a place's name from a bare URL's domain/slug when no notes or screenshot are attached (a URL alone isn't reliable evidence), and a hint under the link field tells the user the same thing.
- **No native dialogs.** Every `alert()`/`confirm()`/`prompt()` (9 call sites) replaced with an in-page toast or modal, reusing the existing modal visual style.
- **Zero third-party page dependencies.** Leaflet 1.9.4's JS and CSS are now inlined directly into `index.html` (previously loaded from cdnjs). Google Fonts removed; replaced with a system font stack that includes PingFang SC and Microsoft YaHei for Chinese text.
- **Category rename.** The "Must-go" browse category is now labeled "Attractions"; any pin previously saved with the old category name is migrated automatically on load.
- **Sample trip.** A "Sample trip" button loads 6 Nominatim-verified real Toronto Old Town landmarks into a brand-new trip (never auto-loaded, never touches existing trips), so the app is useful to try even when live search is unreachable.
- **Repo hygiene.** `trip_pin_planner.html`, previously a byte-identical duplicate of `index.html` at the repository root, has been archived (moved to `archive/`) rather than left as a second apparent production entry file. Confirmed neither file ever referenced the Google Maps JavaScript API or any other Google Maps API — both use Leaflet + OpenStreetMap.
- **Secrets audit.** No API keys or other secrets found anywhere in the working tree or the full git history.

## Success Metrics

Because this has no users yet beyond Rebecca and friends, "success" for this phase is closer to validated learning than growth metrics. Revisit and tighten these once there's a live friend-group cohort.

**Leading indicators (check after each real trip used with the tool)**
- Time from opening the tool to having a finished, ready-to-use plan for a single day (target: under 15 minutes) — not yet measured
- Number of pins that have notes/links/images attached vs. left empty (target: >80% of pins have some attached research by the time the trip happens)
- Whether the route-ordering feature is used at all per trip (target: used in every multi-pin trip)

**Lagging indicators (once used across multiple trips)**
- Whether Rebecca and her friend group choose this over their old workflow (PPT/itinerary doc/back-and-forth map app) for a second and third trip, unprompted
- Qualitative feedback: does it actually replace the "go back and forth between apps" pain point, or just add a fifth app to the mix?

**Measurement method:** informal for now — a short note/debrief after each trip. No analytics instrumentation planned until there's a case for building past the friend-group phase.

## Open Questions

- Should trips be scoped per-device (local storage) or per-account from the start of the friend-sharing phase? (engineering — affects P1 shareable-link design)
- What's the actual friend-group size we're designing sharing/permissions for — 2-3 close friends, or larger groups of 6-8? (product/Rebecca — changes whether simple link-sharing is enough or real accounts are needed)
- If this does move toward public launch later, is the intent a hobby/portfolio project or an actual startup? (Rebecca — this materially changes whether P2 items like a managed AI assistant and accounts are worth investing in, and whether monetization gets revisited)
- Is there a specific upcoming trip that could serve as the first real "dogfood" test? (Rebecca — would give this PRD a concrete validation moment instead of an open-ended timeline)
- Is the app's dependency stack (OpenStreetMap tiles, Nominatim, Overpass, the OSRM-based routing service) reliably reachable from inside mainland China? Not yet tested as of v0.1.1 — affects how much the China-market use case can be relied on without native Amap/Baidu integration (P2).
- Should trip deletion be added before or alongside the P1 route-constraints work? Currently there's no way to remove a trip once created.

## Timeline Considerations

No hard deadline — this is intentionally open-ended and driven by learning, not a launch date.

**Suggested phasing:**
- **Phase 0 (done):** Personal prototype — pin, attach 攻略, AI-assisted parsing, automatic route ordering with manual override, four-way nav redirect, categories, multi-day/multi-trip. Stabilized in v0.1.1 (see Implementation Notes above). Currently usable by Rebecca alone.
- **Phase 1 (next):** P1 requirements — daily time budget + locked-pin removal suggestions, collaborative planning (accounts + shared backend), UI/UX polish — enough to use with friends on a real trip. Trigger to start: Rebecca has used the current prototype on at least one real trip solo and confirmed the core loop (pin → attach → order → navigate) actually removes the app-switching pain.
- **Phase 2:** Use with the friend group across 2+ real trips (per Goal 4). Only after this phase produces a clear "yes, keep building" signal should P2 (managed AI assistant, accounts, native app, potential product/launch direction) be scoped in a follow-up PRD.

---

*Next possible artifacts: a scoped Phase 1 spec for the time-budget + locked-pin feature, or an engineering breakdown of what a lightweight shared backend for collaborative planning would require.*
