# Someday review

`{{WIKI}}/someday/` holds things I *should* do but aren't urgent — parked deliberately, not brainstorms. Each page is one item, with `status: open | done | dropped`.

Difference from `{{WIKI}}/ideas/`:
- **Ideas** — brainstorming. May or may not happen.
- **Someday** — commitments to act, just not now.

Both share the same 3-state enum (`open | done | dropped`). `done` and `dropped` pages are unlisted from `index.md`; find them via folder browse.

## Auto-surface (proactive)

At session start (or first relevant moment), if any someday page has `status: open` and `updated:` is more than ~90 days behind today:

> "You have N someday items not reviewed in 3+ months. Want a someday review?"

One line, dismissible. Don't deep-dive unless I say yes.

## Full review (on request)

When I say "someday review" or accept the auto-surface:

1. Read all pages in `{{WIKI}}/someday/`.
2. List `open` items with one-line summaries and days-since-update.
3. For each stale item (>90 days), ask one question: still want it? Done? Drop?
4. Don't change status silently — wait for my answer per item, then update `status:` and `updated:`.
5. If I want to advance one, help me think through the next concrete step and append to that page's Notes.

## Page schema

Someday pages follow the standard frontmatter plus `status: open | done | dropped`. Body sections:

- **What** — concrete action, one paragraph.
- **Why** — motivation; links to relevant ideas/syntheses/concepts.
- **Next step** — first concrete step (when ready to do it). Often "wait until X" if blocked.
- **Notes** — append-only working notes as I make progress.

## Materialization

Create a `{{WIKI}}/someday/<slug>.md` page when I explicitly mark something as someday — either by writing `[[someday-X]]` with intent to track it, or by saying "park this as someday". Unlike ideas, no ≥2-reference threshold: someday items are explicit commitments. The first mention is enough.

When extracting a someday item that was previously embedded in another page (e.g., as an action bullet inside a `{{WIKI}}/ideas/` page), replace the embedded content with a cross-link (`[[someday/<slug>|<slug>]]`) so the original page stays clean and the someday list is canonical.
