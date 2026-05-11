---
name: review
description: Review the wiki. Dispatcher for four review types — Now (default), Someday, Lint, Tour. Trigger on "review", "now review", "update now", "refresh now page", "someday review", "review someday", "what's in someday", "lint", "lint the wiki", "wiki health check", "tour", "wiki tour", "what have I forgotten". Also auto-surface staleness nudges (see below).
---

# Review

A single entry point for reviewing the wiki. Routes to one of four reviews based on what I said.

## Routing

Pick the route from my phrasing. If ambiguous, ask — defaulting to **Now**.

| Route | Triggers | Sub-file |
|---|---|---|
| **Now** (default) | "review", "now review", "update now", "refresh now page" | `now.md` |
| **Someday** | "someday review", "review someday", "what's in someday" | `someday.md` |
| **Lint** | "lint", "lint the wiki", "wiki health check" | `lint.md` |
| **Tour** | "tour", "wiki tour", "what have I forgotten" | `tour.md` |

When I type a bare `review` (or invoke this skill with no clear keyword), ask:

> "Which review? **Now** (default), Someday, Lint, or Tour?"

One question, one line. If I just say "go" or hit enter, run **Now**.

Once the route is decided, read the matching sub-file in this skill's directory and follow it.

## Auto-surface (proactive nudges)

These are one-line nudges, not full reviews. Surface at most once per session, dismissible.

- **Now staleness** — if `{{WIKI}}/now.md`'s `updated:` is more than ~3 weeks behind today, offer a Now review.
- **Someday staleness** — if any `{{WIKI}}/someday/` page with `status: open` has `updated:` more than ~90 days behind today, say: *"You have N someday items not reviewed in 3+ months. Want a someday review?"*

Don't deep-dive unless I say yes.

## What this skill does NOT do

- Ingest content → `wiki-ingest`
- Answer questions about vault contents → `wiki-query`
