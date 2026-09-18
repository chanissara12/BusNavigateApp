---
id: 006
title: Decide ranking and zero/multiple-match messaging rules
type: grilling
status: closed
assignee: chanissara.s
blocked_by: [004]
---

## Question

Once the core screen prototype has been reacted to, decide: how multiple matching
direct routes should be ranked/ordered, and what the app says when no direct route
exists between the resolved origin and destination stops.

## Resolution

Settled via grilling with the user on 2026-09-18:

- **Ranking**: sort by total walking distance (origin→boarding stop plus
  alighting stop→destination), ascending. Show every matching route, no cap/no
  "load more" — direct-route counts at a single stop are small in practice.
- **Zero-match messaging**: show "ไม่พบรถเมล์สายตรงไปจุดหมายนี้" plus a suggestion
  to try a nearby destination or use another tool (e.g. Google Maps) for
  transfer-based routing, since transfers are out of scope for this app.
- **Far-stop case**: no special warning/badge for long walking distances — the
  walking-distance figure already shown on each result card communicates this.
