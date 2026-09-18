---
id: 002
title: Compare geocoding + map-tile options for Thai addresses on a free tier
type: research
status: closed
assignee: null
blocked_by: []
---

## Question

The MVP needs (a) a text search that resolves a Thai place name/address to
coordinates, and (b) a map widget the user can pin a destination on. Compare, for a
free/low-cost solo project:

- **Longdo Map API** — geocoding + map tiles, quality for Thai place names, pricing
  tiers, rate limits, ToS for personal/free use.
- **OpenStreetMap + Nominatim** (self-hosted or public instance) — free, but check
  usage-policy rate limits and geocoding quality for Thai addresses/place names.
- **Google Maps Platform** (Places/Geocoding API) — free-tier credit, cost after that,
  quality baseline for comparison.

Report which option(s) are actually viable free/cheap for a hobby project at low
request volume, and flag any that are disqualified by ToS or quality for Thai text.
Pure research — do not pick one, that's a separate decision ticket.

## Resolution

Full findings: [research/geocoding-map-options.md](../research/geocoding-map-options.md)

- **None of the three options are disqualified** for a low-volume solo/hobby project.
- **Longdo Map API**: free up to 100,000 requests/month (60 req/min cap), ToS explicitly
  allows personal/non-commercial use, no credit card required. Best Thai place-name
  fidelity of the three; smaller ecosystem/mostly-Thai docs.
- **OSM + Nominatim (public)**: free, no signup, no card — but ToS caps requests at
  1/sec (4/min for scheduled/long-running scripts), forbids autocomplete-style search
  and bulk/systematic querying, and requires caching results. Thai data quality is
  contributor-dependent (patchier for sois/informal addresses); self-hosting is an
  option if limits become blocking but adds real ops burden.
- **Google Maps Platform**: 10,000 free requests/month per API (Geocoding/Places),
  $5/1,000 after that — but requires a Google Cloud billing account with a valid
  credit card on file even to stay within the free tier, which is a real friction/risk
  point for a no-budget hobby project. Quality baseline is strong, including for Thai
  addresses.
- A Thai-address-geocoding accuracy study found rooftop-level positional accuracy is
  imperfect across all major geocoders, and Thai postal codes only resolve to the
  province level — a caveat regardless of provider chosen.
