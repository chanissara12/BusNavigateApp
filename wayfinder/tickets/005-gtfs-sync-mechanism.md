---
id: 005
title: Decide GTFS sync/update mechanism and cadence
type: grilling
status: open
assignee: null
blocked_by: [001]
---

## Question

Given the confirmed shape and update frequency of the bangkok-gtfs feed, decide how
the app syncs it (manual re-download vs. scheduled job), how often, and where the
processed data lives (e.g., baked into a static build vs. a small database).
