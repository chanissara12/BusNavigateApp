---
id: 001
title: Confirm bangkok-gtfs data shape supports direct-route stop-pair queries
type: research
status: closed
assignee: null
blocked_by: []
---

## Question

The MVP needs to answer "which bus route(s) pass through stop A, then later stop B,
in that direction (no transfers)?" using the free [asiripanich/bangkok-gtfs](https://github.com/asiripanich/bangkok-gtfs)
feed (a mirror of OTP's Namtang GTFS).

Investigate the feed's actual files (`stops.txt`, `routes.txt`, `trips.txt`,
`stop_times.txt`, `shapes.txt` if present) and confirm:

- Can "stop A comes before stop B on the same trip/direction" be determined directly
  from `stop_times.txt` (via `stop_sequence`), or does it require extra processing?
- How are route directions/variants represented (`direction_id`, multiple `trip_id`s
  per route, etc.) — is there a clean way to dedupe to "one row per route+direction"?
- Data freshness: how often is this mirror updated, and how stale can it get?
- Any known data quality issues (missing stops, wrong sequences, duplicate routes)
  reported in the repo's issues/README worth flagging for the spec.

This is pure research — do not design the query or pipeline, just report the facts
the pipeline design ticket will need.

## Resolution

Full findings: [`research/gtfs-data-shape.md`](../research/gtfs-data-shape.md).

- **Stop order**: `stop_times.txt` has `trip_id, arrival_time, departure_time, stop_id, stop_sequence, timepoint`. `stop_sequence` increments cleanly per trip in the sampled data, so "A before B" is a direct filter+compare on `stop_sequence` — no reconstruction needed. Recommend a sanity check (unique/increasing sequence per trip) rather than assuming it holds feed-wide.
- **Direction/variants**: `trips.txt` has `route_id, service_id, trip_id, trip_headsign, direction_id, shape_id, wheelchair_accessible`. Multiple `trip_id`s per `(route_id, direction_id)` mostly come from different `service_id` (calendar) variants, not different physical paths — `shape_id`/`trip_headsign` were stable within `(route_id, direction_id)` in the sample. Dedupe to "one row per route+direction" by grouping on `(route_id, direction_id)` and picking a representative trip; verify at full scale that `shape_id` doesn't diverge within a group (fall back to `(route_id, direction_id, shape_id)` if it does).
- **Freshness**: repo's stated goal is daily snapshots via a GitHub Actions job (~00:36–00:46 UTC), but historical gaps exist (multi-week/month gaps seen in the snapshot history). Treat it as "daily, best-effort" — check the actual latest commit date at build time rather than assuming same-day currency.
- **Known issues**: only 2 open GitHub issues, both repo-hygiene (README improvement, avoiding feed_version-only commits) — no reported data-quality issues (missing stops, bad sequences, duplicate routes). Since this is a thin mirror of an upstream feed, absence of reported issues isn't strong evidence of quality; the pipeline should still do its own basic validation.
- Full standard GTFS static feed present: agency, stops, routes, trips, stop_times, shapes, calendar, calendar_dates, fare_attributes, fare_rules, feed_info, frequencies.
