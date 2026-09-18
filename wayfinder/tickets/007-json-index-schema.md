---
id: 007
title: Design the stop-pair-to-routes JSON index schema and build step
type: grilling
status: closed
assignee: chanissara.s
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

## Resolution

Settled via grilling with the user on 2026-09-18:

- **Index structure**: per-stop, not per-stop-pair. Each stop id maps to a list of
  `(route_id, direction_id, sequence_position)` entries. The client intersects a
  candidate origin stop's list against a candidate destination stop's list at query
  time (same route+direction, origin's position before destination's) — avoids the
  combinatorial blowup of precomputing every stop pair.
- **Candidate stops**: not just the single nearest stop per side — every stop within
  a 500m walking radius of the origin and of the destination is a candidate, and
  results are unioned across all candidate-stop combinations. Guards against missing
  a real direct route just because the single closest stop happens to lack it.
- **Stop data**: kept in a separate `stops.json` (`stop_id → name, lat/lng`), reused
  for map pins; the main index references stops only by id, not by embedding names.
- **Build script**: `scripts/build-gtfs-index.mjs`, runnable both locally and from
  the ticket 005 GitHub Actions workflow, writing `src/data/gtfs-index.json` and
  `src/data/stops.json`, which get committed to `main`.
- **Validation** (feeds ticket 005's fail-loudly-but-don't-block behavior): the
  build fails if the new stop count or route count drops more than 20% versus the
  previous committed snapshot, or if the generated files don't parse as valid JSON.
