---
id: map
title: Bangkok Bus Wayfinder App — Spec
labels: [wayfinder:map]
tracker: local-markdown (see TRACKER.md)
---

## Destination

A **product spec** (no code yet) for a solo-built Thai-language **web app (PWA)** that
tells a user, given their current location and a desired destination, which Bangkok
bus route(s) to take directly (no transfers) — bus number, boarding/alighting stops,
and walking distance to the boarding stop. Real-time tracking, transfers, fares,
English UI, native mobile apps, and non-bus transit are explicitly out of scope for
this destination (see below).

## Notes

- Domain: Bangkok public transit (BMTA + รถร่วม + BMA Feeder buses only).
- Skills every session should consult: `grilling`, `domain-modeling` for decision
  tickets; `prototype` for the UI ticket.
- Standing preference: free/low-cost only, solo-maintained project — prefer options
  with generous free tiers over paid APIs.
- Known base dataset: [asiripanich/bangkok-gtfs](https://github.com/asiripanich/bangkok-gtfs)
  (community mirror of OTP's official "Namtang GTFS" feed, CC BY 4.0, free).
- No public/free real-time bus GPS API was found for Bangkok as of 2026-09-18
  (ViaBus, Longdo, BMTA BUS app all lack documented public APIs) — this is why
  real-time tracking is out of scope for this destination, not just deferred casually.

## Decisions so far

- Destination scope, platform, phasing (this map's Destination section above) —
  settled via grilling on 2026-09-18: MVP is direct-route-only, Thai-only, web PWA,
  GTFS-static-backed, no fares/transfers/real-time.
- [Confirm bangkok-gtfs data shape supports direct-route stop-pair queries](tickets/001-gtfs-data-shape.md):
  `stop_sequence` in `stop_times.txt` directly supports "A before B" checks; dedupe
  trips to one row per `(route_id, direction_id)`; feed updates daily (best-effort,
  gaps possible); no reported data-quality issues but pipeline should self-validate.
- [Compare geocoding + map-tile options for Thai addresses on a free tier](tickets/002-geocoding-map-options.md):
  none of Longdo Map, OSM/Nominatim, or Google Maps Platform are disqualified for a
  low-volume hobby project; Longdo (free to 100k req/month, no card, best Thai
  fidelity) and OSM/Nominatim (free, but bans autocomplete UX, 1 req/sec cap, needs
  caching) are the front-runners since Google requires a billing card even for its
  free tier.
- [Decide recommended tech stack and hosting for the spec](tickets/003-tech-stack-hosting.md):
  Longdo Map API + React/Vite PWA (via `vite-plugin-pwa`) + GTFS precomputed into a
  static client-side JSON index (no backend) + Cloudflare Pages hosting.

## Not yet specified

- Ranking/tie-breaking rules when multiple direct routes match, and messaging for the
  zero-match case, aren't sharp yet — depends on seeing the prototype reaction (see
  ticket 006, blocked on the ticket 004 prototype).

## Out of scope

- Real-time bus position tracking / live ETA — no free public API exists for Bangkok
  buses as of this writing; revisit only if the destination is redrawn to include it.
- Multi-leg transfers between two bus routes — real routing-graph complexity, ruled
  out to keep the MVP shippable by a solo developer; confirmed with user 2026-09-18.
- Fare estimates — fare data is not reliably available/stable; deferred indefinitely.
- English UI / bilingual support — Thai-only for MVP.
- Native mobile app (iOS/Android) — web PWA only.
- Non-bus transit (BTS/MRT/boat) and multi-modal routing — bus-only scope.
