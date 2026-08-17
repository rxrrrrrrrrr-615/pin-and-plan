# Pin & Plan — Product Requirements Document

**Status:** Draft v1 · **Owner:** Rebecca · **Stage:** 0-to-1, pre-launch prototype in personal use

---

## Problem Statement

Travel planning today is scattered across too many disconnected tools. People discover places to go through 小红书, blogs, or Google, save them as scattered bookmarks or screenshots, then manually rebuild that list into a PPT, itinerary doc, or spreadsheet. Separately, they open a map app to figure out which order to visit things in, then switch to yet another app to actually book tickets or navigate. This is especially painful for "citywalk" style trips, common on girls' trips, where the plan is a loose sequence of small stops (a café, then a boutique, then a photo spot, then a restaurant) rather than a few big-ticket attractions. There is no single place to both decide *where* to go and *in what order*, and no existing tool lets a pinned place carry its own personal research (a pasted link, a screenshot, a note) as a ready-to-use guide. The cost of not solving this is hours of manual, repetitive replanning for every trip, and a plan that lives in three or four different apps at once instead of one.

This is grounded in Rebecca's own recurring pain point across multiple trips, not yet validated with a broader user sample — validating it with friends is part of this plan (see Phasing).

## Goals

1. Cut the time to go from "list of places I want to visit" to "an ordered, ready-to-use plan" to under 15 minutes for a single-day citywalk.
2. Every pinned place carries its own attached research (link, note, or image) so the plan is self-contained — no bouncing back to 小红书/blogs to remember why a place was saved.
3. Works the same way regardless of country — a trip in Toronto and a trip in Chengdu use the same tool, with navigation handoff to whichever map app (Google Maps, Amap, Baidu Maps) is actually usable in that country.
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
- I want the tool to suggest an efficient walking order for my pinned stops, so that I don't have to manually reason about which store is closer to which restaurant.
- I want to see a rough walking distance and time for my planned route, so that I know if I'm overplanning a single day.
- I want to tap a pin and open it directly in Google Maps, Amap, or Baidu Maps, so that I can actually get walking directions using whichever app works in the country I'm in.
- I want my pins and notes to persist when I close and reopen the tool, so that I don't lose my planning work.

**As a friend invited to help plan or follow the trip,**
- I want to view (and ideally edit) the same trip plan the organizer built, so that we're planning together instead of over text messages.
- I want to see whose 攻略/notes are attached to which pin, so that I know the source of a recommendation, especially if multiple friends contributed spots.

**As either user, on the day of the trip,**
- I want to open the trip plan on my phone and quickly get walking directions to the next stop, so that I'm not fumbling between apps mid-walk.
- I want to reorder or remove a pin on the fly if plans change (a shop is closed, we're tired, we found something better), so that the plan flexes with the actual day rather than being a fixed document.

**Edge cases**
- As a user, if I search for a place with no results or a typo, I want a clear "no results" state rather than a silent failure.
- As a user, if the route-optimization service is temporarily unavailable, I still want to see a straight-line route and distance estimate rather than nothing.
- As a user, if I have zero or one pin, I want the tool to not attempt route optimization and instead prompt me to add more pins.

## Requirements

### Must-Have (P0) — validated in the current prototype
- [x] Drop a pin by clicking the map or searching a place name (free geocoding, works in any country)
- [x] Attach free-text notes, a pasted link, and an image to each pin
- [x] Auto-generate a suggested visiting order (nearest-neighbor route heuristic) with one tap
- [x] Draw an actual walking route line with distance and time estimate, with a straight-line fallback if the routing service is down
- [x] One-tap redirect from any pin to Google Maps, Amap, or Baidu Maps for real navigation
- [x] Data persists locally between sessions (browser local storage)
- [x] Works the same in any country — no hardcoded region or default location; either resumes your last view or starts from your current location

**Acceptance criteria (example, for route optimization):**
- Given at least 2 pins exist on the map
- When the user taps "Optimize walking order"
- Then the pin list re-orders into a shorter total walking route, a route line is drawn on the map, and the estimated distance/time updates
- Given the routing service (OSRM) is unreachable
- When optimization is triggered
- Then a straight-line route and rough distance estimate are still shown, and no error is surfaced to the user

### Nice-to-Have (P1) — needed before showing this to friends
- [ ] Named trips: create multiple separate trips instead of one flat pin list, so a Canada trip and a China trip don't collide
- [ ] Shareable link: generate a link a friend can open to view/edit the same trip (requires moving off pure local-storage to some lightweight shared backend)
- [ ] Day-grouping: assign pins to Day 1 / Day 2 / etc. for multi-day trips, with route optimization run per day
- [ ] Basic account/sync so a trip isn't lost if local storage is cleared or the user switches devices
- [ ] Mobile-friendly layout pass (current prototype is usable but optimized for desktop-width screens)

### Future Considerations (P2) — explicitly deferred, but should not be architecturally blocked
- [ ] Real AI chat assistant (currently a rule-based stub) that can answer "should I visit X before Y", ingest a pasted 攻略 and auto-suggest which parts to pin, or summarize a long blog post into pin-ready notes — requires wiring an actual LLM API and a backend to hold the key
- [ ] Native mobile app for offline use and better on-the-go navigation handoff
- [ ] Native Amap/Baidu search integration (better China place-search quality than free OSM/Nominatim data)
- [ ] Opening-hours and crowd-time awareness when suggesting route order
- [ ] Public trip templates or discovery (explicitly non-goal for now, but don't design the data model in a way that makes this impossible later)

## Success Metrics

Because this has no users yet beyond Rebecca and friends, "success" for this phase is closer to validated learning than growth metrics. Revisit and tighten these once there's a live friend-group cohort.

**Leading indicators (check after each real trip used with the tool)**
- Time from opening the tool to having a finished, ready-to-use plan for a single day (target: under 15 minutes)
- Number of pins that have notes/links/images attached vs. left empty (target: >80% of pins have some attached research by the time the trip happens)
- Whether the route-optimize feature is used at all per trip (target: used in every multi-pin trip)

**Lagging indicators (once used across multiple trips)**
- Whether Rebecca and her friend group choose this over their old workflow (PPT/itinerary doc/back-and-forth map app) for a second and third trip, unprompted
- Qualitative feedback: does it actually replace the "go back and forth between apps" pain point, or just add a fifth app to the mix?

**Measurement method:** informal for now — a short note/debrief after each trip. No analytics instrumentation planned until there's a case for building past the friend-group phase.

## Open Questions

- Should trips be scoped per-device (local storage) or per-account from the start of the friend-sharing phase? (engineering — affects P1 shareable-link design)
- What's the actual friend-group size we're designing sharing/permissions for — 2-3 close friends, or larger groups of 6-8? (product/Rebecca — changes whether simple link-sharing is enough or real accounts are needed)
- If this does move toward public launch later, is the intent a hobby/portfolio project or an actual startup? (Rebecca — this materially changes whether P2 items like AI chat and accounts are worth investing in, and whether monetization gets revisited)
- Is there a specific upcoming trip that could serve as the first real "dogfood" test? (Rebecca — would give this PRD a concrete validation moment instead of an open-ended timeline)

## Timeline Considerations

No hard deadline — this is intentionally open-ended and driven by learning, not a launch date.

**Suggested phasing:**
- **Phase 0 (done):** Personal prototype — pin, attach 攻略, route optimize, multi-market nav redirect. Currently usable by Rebecca alone.
- **Phase 1 (next):** P1 requirements — named trips, shareable link, day-grouping — enough to use with friends on a real trip. Trigger to start: Rebecca has used the current prototype on at least one real trip solo and confirmed the core loop (pin → attach → optimize → navigate) actually removes the app-switching pain.
- **Phase 2:** Use with the friend group across 2+ real trips (per Goal 4). Only after this phase produces a clear "yes, keep building" signal should P2 (real AI assistant, accounts, native app, potential product/launch direction) be scoped in a follow-up PRD.

---

*Next possible artifacts: a scoped Phase 1 spec (shareable trips + day-grouping), or an engineering breakdown of what a lightweight shared backend would require.*
