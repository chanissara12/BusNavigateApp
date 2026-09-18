# Local Markdown Tracker (Wayfinder)

No hosted issue tracker (GitHub/GitLab/Jira) is configured for this repo, so Wayfinder
uses this folder as the issue tracker. This file documents the "Wayfinding operations"
convention so any session can read/write it consistently.

## Layout

- `wayfinder/map.md` — the map issue (labelled `wayfinder:map` in its frontmatter).
- `wayfinder/tickets/<id>-<slug>.md` — one file per child ticket.
- `wayfinder/research/<slug>.md` — findings captured by research tickets (linked from
  the ticket, not pasted into it).

## Ticket frontmatter

```yaml
---
id: 003                        # stable numeric id, assigned at creation, never reused
title: Decide tech stack and hosting
type: research | prototype | grilling | task
status: open | closed
assignee: null | "<name>"      # claim = set this before starting work
blocked_by: [001, 002]         # numeric ids of tickets that must be closed first
---
```

## Wayfinding operations

- **Frontier query**: tickets with `status: open`, `assignee: null`, and every id in
  `blocked_by` pointing at a ticket with `status: closed`.
- **Claim**: edit the ticket's `assignee` field to the claiming session/dev, before any
  work.
- **Blocking**: native to this tracker via the `blocked_by` list (ids only, this repo
  has no cross-repo tickets). A ticket is unblocked when every id it lists is closed.
- **Resolve**: append a `## Resolution` section to the ticket body with the answer,
  set `status: closed`, then update `wayfinder/map.md`'s "Decisions so far" with a
  one-line gist + link.
- **Research findings**: saved as their own file under `wayfinder/research/`, linked
  from the ticket's `## Resolution` (or a `## Context` pointer) — never pasted into
  the map.
