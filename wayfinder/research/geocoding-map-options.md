# Research: Geocoding + Map-Tile Options for Thai Addresses (Free/Low-Cost, Solo Hobby Project)

Related ticket: [002-geocoding-map-options.md](../tickets/002-geocoding-map-options.md)

Date: 2026-09-18

## Summary Table

| Option | Free tier | Rate limit | Thai place-name quality | ToS for hobby/personal use |
|---|---|---|---|---|
| Longdo Map API | 100,000 requests/month, all services combined, at no cost | 60 requests/minute | Purpose-built for Thailand; strong sub-district/district/Thai-script handling | Explicitly allows free use up to threshold; personal/non-commercial use is permitted |
| OSM + Nominatim (public) | Free, no signup | 1 req/sec absolute max; 4 req/min for long-running or scheduled scripts; must cache; no autocomplete/bulk/scraping | Decent for well-mapped urban areas (Bangkok core), but data quality is OSM-contributor-dependent — weaker for informal sois, new developments, or postal-code-based lookups. Academic study found Thai geocoders in general suffer poor positional (rooftop-level) accuracy, and Thai postal codes only resolve to province level, not lower admin units | Public instance ToS forbids autocomplete-as-you-type UX, heavy/bulk use, and reselling; must self-host if these are needed |
| Google Maps Platform (Geocoding/Places) | 10,000 free "events"/API/month (Essentials SKU) — replaced the old universal $200/month credit in Feb/Mar 2025 | Governed by quota + billing, not a hard rate cap | Generally considered a strong quality baseline, incl. for Thai addresses (used as one of the higher-performing comparators in the Thai-geocoding accuracy study) | Requires a Google Cloud Billing account with a valid credit card on file even to stay within the free tier; usage beyond 10k/month is billed at $5/1,000 requests (Geocoding) |

## 1. Longdo Map API (map.longdo.com)

- **What it offers**: Forward/reverse geocoding, place search, and map tiles, purpose-built for Thailand — noted for strong Thai sub-district/district/postal fidelity and Thai-script place-name search that general-purpose geocoders often mishandle.
- **Free tier / pricing**: Per the official Terms of Use (https://map.longdo.com/api/terms/), the API is free of charge up to the "Longdo Map API Free Usage Threshold," currently **100,000 web-service requests per month across all services**, capped at **60 requests/minute**. Beyond that, usage is pay-as-you-go (monthly/annual packages); exact paid-tier pricing wasn't published on the marketing landing page and requires visiting the dedicated pricing page (https://map.longdo.com/products/) or contacting sales.
- **ToS for personal/free use**: The terms explicitly permit personal and non-commercial use, and getting an API key is free (self-serve "Longdo ID" registration). No indication of a "commercial use only" restriction blocking a solo hobby project.
- **Fit for this project**: At a hobbyist's low request volume, 100k/month and 60/min are very generous headroom — effectively unlimited for MVP purposes. This is the strongest fit purely on Thai-language/place-name quality plus a comfortable free quota, though the SDK/community ecosystem is smaller than Google's or OSM's (fewer client libraries, docs mostly Thai-first).
- **Sources**: https://map.longdo.com/api/terms/, https://map.longdo.com/products/, https://api.longdo.com/map/doc/rest.php

## 2. OpenStreetMap + Nominatim

- **Public instance (nominatim.openstreetmap.org)**: Completely free, no API key or signup. Governed by the OSMF Nominatim Usage Policy (https://operations.osmfoundation.org/policies/nominatim/):
  - Absolute max **1 request/second**; scripts that run continuously or on a schedule are limited to **4 requests/minute**.
  - Must set a valid HTTP Referer or custom User-Agent identifying the app (default library user-agents are rejected/may be blocked).
  - **Must cache results** on your side; no re-querying identical lookups.
  - **Forbidden**: autocomplete-style/type-ahead search, systematic/bulk queries (e.g., grid reverse-geocoding, dataset scraping), and reselling/operating a geocoding-reseller service on top of it.
  - Bulk geocoding is tolerated only for small one-off jobs, single-threaded, single machine.
- **Self-hosting**: An option if the public instance's rate limits or the autocomplete restriction become blocking, but it requires standing up and maintaining a full Nominatim + OSM planet/region data stack (non-trivial ops burden for a solo dev — not "free" in effort terms even though there's no monetary licensing cost).
- **Thai geocoding quality**: OSM data quality in Thailand is contributor-dependent — reasonable in well-mapped central Bangkok, patchier for sois, informal addresses, and newer developments. An academic study on Thai-text-address geocoding (ph01.tci-thaijo.org / EASR journal) found that Thai postal codes resolve only to the province level with no lower-level geospatial meaning, which trips up geocoders that lean on postal code, and that even the better-performing commercial services didn't reach reliable rooftop-level accuracy for Thai addresses generally. Nominatim wasn't the specific top performer in that comparison (Google/Bing/Yahoo/OpenCage scored higher on match rate).
- **Fit for this project**: Usable for a low-volume MVP as long as the UI avoids live autocomplete-while-typing (a "search on submit" pattern would need to respect the 1 req/sec and ideally debounce/cache) and results are cached client- or server-side. The ToS restriction on autocomplete is the main practical friction point for a "pin a destination" search box.
- **Sources**: https://operations.osmfoundation.org/policies/nominatim/, https://ph01.tci-thaijo.org/index.php/easr/article/view/140887

## 3. Google Maps Platform (Geocoding / Places API)

- **Free tier**: As of the pricing overhaul (effective ~March 2025), the old universal $200/month credit was retired. Each Core Services SKU now gets its own free monthly allowance; Geocoding (and most Essentials-tier SKUs, including Places) get **10,000 free requests/month**.
- **Pricing beyond free tier**: Geocoding API is **$5.00 per 1,000 requests** after the free 10k. Places API pricing varies by endpoint/field mask and is typically pricier per call than Geocoding.
- **Requirement**: A Google Cloud **Billing account with a valid credit card is mandatory** to even generate/activate an API key, regardless of whether usage stays within the free tier. This is a meaningful friction/risk point for a hobby project (card on file, possibility of surprise charges if usage spikes or a key leaks/gets abused).
- **Quality**: Considered a strong baseline for geocoding quality generally, including for Thai addresses in the comparative study referenced above (Google was among the higher match-rate performers).
- **Fit for this project**: At true low hobby volume (well under 10k/month), Google is free in practice, and quality is good — but it's the only option requiring a credit card on file and carries real cost risk if volume grows or a key is exposed publicly (a real concern for an open client-side hobby app).
- **Sources**: https://developers.google.com/maps/documentation/geocoding/usage-and-billing, https://developers.google.com/maps/billing-and-pricing/faq, https://developers.google.com/maps/billing-and-pricing/pricing

## Flags / Disqualifiers

- **None of the three are outright disqualified** by ToS for a low-volume personal/hobby project.
- **Nominatim (public)**: not disqualified, but its ToS explicitly forbids autocomplete-style UX and bulk/systematic querying — this constrains *how* the search box can be built (no live-as-you-type suggestions against the public endpoint) and requires caching.
- **Google Maps**: not disqualified, but requires a credit-card-backed billing account even for free-tier-only usage — a real adoption friction/risk for a no-budget solo project, distinct from the other two which need no card at all.
- **Longdo**: not disqualified; only caveat is smaller ecosystem/English-language documentation and unpublished paid-tier pricing (not relevant at hobby volume since the free 100k/month threshold is generous).
- **Quality caveat (all three)**: A dedicated academic study on Thai-address geocoding found that positional (rooftop-level) accuracy is imperfect across all major services, and Thai postal codes are not useful as a geocoding signal below the province level — worth keeping in mind regardless of which provider is chosen.
