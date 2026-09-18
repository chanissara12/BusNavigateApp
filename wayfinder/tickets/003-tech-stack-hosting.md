---
id: 003
title: Decide recommended tech stack and hosting for the spec
type: grilling
status: closed
assignee: chanissara.s
blocked_by: [002]
---

## Question

The spec should recommend a concrete frontend stack, backend/data-serving approach
(for the synced GTFS data), and a free/low-cost hosting target, consistent with the
map/geocoding option chosen in the map+geocoding ticket. Resolve via grilling with
the user: what stack fits a solo-maintained PWA, and where does it deploy for free?

## Resolution

Settled via grilling with the user on 2026-09-18:

- **Geocoding/map provider**: Longdo Map API (free to 100k req/month, no card
  required, best Thai place-name fidelity).
- **Frontend**: React + Vite (SPA), PWA via `vite-plugin-pwa`.
- **Data serving**: no backend — GTFS is processed at build time into a
  precomputed JSON index (stop-pair → matching direct routes), baked into the
  static build, queried entirely client-side.
- **Hosting**: Cloudflare Pages (unlimited free bandwidth, auto-deploy from the
  GitHub repo already pushed at chanissara12/BusNavigateApp).
