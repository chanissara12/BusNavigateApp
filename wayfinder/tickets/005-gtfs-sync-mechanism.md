---
id: 005
title: Decide GTFS sync/update mechanism and cadence
type: grilling
status: closed
assignee: chanissara.s
blocked_by: [001]
---

## Question

Given the confirmed shape and update frequency of the bangkok-gtfs feed, decide how
the app syncs it (manual re-download vs. scheduled job), how often, and where the
processed data lives (e.g., baked into a static build vs. a small database).

## Resolution

Settled via grilling with the user on 2026-09-18:

- **Cadence**: weekly sync (bus route structure changes rarely; more frequent sync
  adds risk of hitting a transient bad upstream snapshot for no real benefit).
- **Mechanism**: a scheduled GitHub Actions workflow (cron) pulls the latest
  bangkok-gtfs data, runs the build script (schema/internals decided separately in
  ticket 007), and commits + pushes to `main` if the data changed — which triggers
  Cloudflare Pages' existing auto-deploy.
- **Staleness/bad-data handling**: silently keep serving the last-known-good build
  (never block deploys), but run a validation step in the workflow that fails loudly
  (triggering GitHub's automatic failure-notification email) if the upstream feed is
  stale or looks malformed — self-alerting without needing an on-call process.
