---
id: 007
title: Design the stop-pair-to-routes JSON index schema and build step
type: grilling
status: open
assignee: null
blocked_by: []
---

## Question

Ticket 003 settled that GTFS data is precomputed at build time into a static JSON
index (stop-pair → matching direct routes) and queried entirely client-side, with no
backend. Ticket 001 confirmed `stop_sequence` in `stop_times.txt` supports "A before
B" checks directly, and trips dedupe to one row per `(route_id, direction_id)`.

Decide the concrete shape of that index and the build step that produces it:

- What's the lookup key — nearest-N stop ids around origin/destination, or something
  coarser (e.g. geohash bucket) to keep the index small enough to ship to the client?
- What does each index entry contain (route id, direction, boarding stop, alighting
  stop, display name/number) — enough for the UI in ticket 004 to render a result
  without further lookups?
- Where does the build step live (a script run manually / in CI before each Cloudflare
  Pages deploy) and what triggers a rebuild when the upstream GTFS feed updates?
