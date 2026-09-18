# GTFS Data Shape — asiripanich/bangkok-gtfs

Research date: 2026-09-18
Source: https://github.com/asiripanich/bangkok-gtfs (README, `data/gtfs/` file listing, issues, and raw CSV samples fetched via GitHub raw content on the `main` branch).

## 1. Can stop-order-on-a-trip be read directly from `stop_times.txt`?

Yes, directly — no extra processing needed beyond a sort.

Confirmed columns in `stop_times.txt`: `trip_id, arrival_time, departure_time, stop_id, stop_sequence, timepoint`.

Sample rows for `trip_id = 5`:

```
trip_id, arrival_time, departure_time, stop_id, stop_sequence, timepoint
5, 00:00:00, 00:00:00, 13,    1, 0
5, 00:01:00, 00:01:30, 12,    2, 0
5, 00:04:30, 00:05:00, 11,    3, 0
5, 00:07:00, 00:07:30, 10,    4, 0
5, 00:09:30, 00:10:00, 9,     5, 0
5, 00:12:00, 00:12:30, 13844, 6, 0
...
```

`stop_sequence` increments monotonically (1, 2, 3, …) per `trip_id`, exactly per the GTFS spec. To answer "does stop A come before stop B on this trip," it's enough to filter `stop_times` to a given `trip_id` and compare `stop_sequence` values for stop A vs stop B (lower sequence = earlier). No gap-filling, timestamp inference, or reconstruction of order from times is required — the sequence numbers are already correct and present for every stop time row in the sample.

Caveat: this was verified on one sample trip; it wasn't feasible to bulk-validate every trip_id in the feed for gaps/duplicates in `stop_sequence`, so the pipeline design should still include a cheap sanity check (e.g., assert sequences are unique and increasing per trip) rather than assuming it blindly across the whole feed.

## 2. How are route directions/variants represented? Is there a clean dedupe path?

`trips.txt` columns: `route_id, service_id, trip_id, trip_headsign, direction_id, shape_id, wheelchair_accessible`.

Sample:

```
route_id, service_id, trip_id, trip_headsign,                          direction_id, shape_id
2,        1,          5,       "บางหว้า;Bang Wa",                      0,            3
2,        1,          6,       "สนามกีฬาแห่งชาติ;National Stadium",     1,            2
2,        2,          7,       "บางหว้า;Bang Wa",                      0,            3
2,        2,          8,       "สนามกีฬาแห่งชาติ;National Stadium",     1,            2
181,      1,          9,       "ท่านนทบุรี;Nonthaburi Pier",            0,            9
181,      1,          10,      "วัดราชสิงขร;Wat Ratchsingkhon Pier",    1,            8
```

Findings:

- `direction_id` (0/1) is populated and consistent with `trip_headsign` — same route+direction always shares the same headsign and `shape_id`.
- Fan-out is by `service_id` (calendar/schedule variant, e.g. weekday vs weekend service), not by route geometry. Route 2, direction 0 appears as trip_id 5 (service_id 1) and trip_id 7 (service_id 2) — two trips, same physical route/direction, different service calendars.
- **Dedupe recipe**: group `trips.txt` by `(route_id, direction_id)` and take any one representative `trip_id` (or the one with the most common `shape_id`/headsign if variants disagree) to get "one row per route+direction." In the sample this is clean — `shape_id` and `trip_headsign` are stable within a `(route_id, direction_id)` group — but this was only checked on ~14 rows, so the pipeline should verify at full-feed scale that `shape_id`/`headsign` don't diverge within a `(route_id, direction_id)` group (i.e., that a route+direction doesn't secretly have multiple distinct physical variants sharing one `direction_id`). If they do diverge, dedupe would need to key on `(route_id, direction_id, shape_id)` instead.

## 3. Data freshness

- The repo's stated purpose (per its README): "make daily snapshots of Bangkok GTFS accessible to anyone." It automates a daily capture via a GitHub Actions workflow (badge referenced as "snap-gtfs"), running roughly around midnight UTC (observed snapshot timestamps ~00:36–00:46 UTC in the commit/snapshot history).
- History shows gaps in the daily cadence (e.g., a gap between late March and mid-January, and another between October and January, in the 2022–2023 timeframe visible in the README's snapshot table) — so "daily" is the intent/design, not a guarantee; the pipeline should not assume every calendar day has a snapshot and should tolerate missing days.
- The project's own README frames itself as starting later than ideal ("Better late than never!"), i.e. it's a lightweight community mirror, not an SLA-backed data service.
- Practical implication for staleness: treat "freshness" as "as of the most recent successful daily GitHub Actions run," and check the actual latest commit date in `data/gtfs/` at build time rather than assuming same-day currency.

## 4. Known data-quality issues

- The repository's GitHub Issues currently show only 2 open issues, both about repo/process hygiene, not data quality:
  - #1 "Improve README" (Oct 2022) — documentation request.
  - #2 "Don't commit `feed_version` only changes" (Apr 2023) — a request to avoid noisy commits when only the feed_version metadata changes between snapshots.
- No open or closed issues were found specifically reporting missing stops, wrong `stop_sequence` ordering, or duplicate routes.
- This is a source-mirror repo (it re-publishes an upstream feed, described elsewhere as sourced from Bangkok's OTP/Namtang GTFS), so any deeper data-quality problems (e.g., stale stop coordinates, orphaned trips, agency-side errors) would originate upstream and likely aren't tracked in this mirror's own issue tracker at all. Absence of issues here is weak evidence of quality, not strong evidence — the spec/pipeline design should budget for its own validation pass (duplicate stop_ids, sequence gaps, orphan trip_ids/route_ids) rather than relying on upstream issue reports.

## Files present in the feed (`data/gtfs/`)

`agency.txt`, `stops.txt`, `routes.txt`, `trips.txt`, `stop_times.txt`, `shapes.txt`, `calendar.txt`, `calendar_dates.txt`, `fare_attributes.txt`, `fare_rules.txt`, `feed_info.txt`, `frequencies.txt` — a standard full GTFS static feed, not a partial extract.

## Summary for the pipeline ticket

- `stop_times.stop_sequence` directly answers "A before B on trip X" with a simple filter + compare — no extra processing needed, just per-trip sanity-checking.
- `trips.direction_id` + `route_id` gives a clean route+direction grouping key; a representative-trip dedupe should work but hasn't been verified at full scale — check for `shape_id`/headsign divergence within groups before trusting it blindly.
- Freshness is "daily, best-effort" via a GitHub Actions snapshot job with observed historical gaps — don't assume the mirror is always same-day fresh; check the actual latest snapshot date at build/query time.
- No documented data-quality issues (missing stops, bad sequences, duplicate routes) exist in the repo's issue tracker, but that's mostly because this is a thin mirror of an upstream feed — plan for the pipeline to do its own basic validation rather than trusting "no issues reported" as a guarantee.
