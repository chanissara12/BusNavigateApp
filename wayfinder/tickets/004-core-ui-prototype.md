---
id: 004
title: Prototype the destination-search → matching-routes screen
type: prototype
status: closed
assignee: chanissara.s
blocked_by: [002]
---

## Question

Build a rough, throwaway UI prototype of the core screen: destination search (text +
map pin) → nearest origin stop → list of direct bus routes with boarding/alighting
stop and walking distance. Use it to react with the user on whether this reads as
"ดูง่าย" (easy to see/scan) before writing it into the spec. Uses whichever
map/search widget was chosen in the geocoding ticket.

## Resolution

Built 3 structurally different variants (list-first, map-first-with-carousel,
big-badge-focus) and reacted with the user. Winning design: the **map section from
Variant B** (shows origin/destination + route line at a glance) **combined with the
list-style result cards from Variant A** (bus number badge, boarding→alighting stop,
walking-distance pill) — rejecting B's horizontal carousel in favor of A's vertically
scannable list.

- Winning combined mock (kept on `main` as a design reference):
  [`prototype/004-core-screen.winner.html`](../../prototype/004-core-screen.winner.html)
- Full 3-variant exploration (throwaway, not on `main`):
  [`prototype/004-core-screen` branch](https://github.com/chanissara12/BusNavigateApp/tree/prototype/004-core-screen)
