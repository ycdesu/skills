---
name: wiki-ingest
description: Ingest sources or inbox fragments into the wiki. Trigger on "ingest [source]", "ingest inbox", "ingest fragments", or "ingest" when inbox contents are short/image items.
---

# Wiki ingest

## Move policy — MUST only move ingested files

A file is **ingested** iff at least one is true:

- A new `{{WIKI}}/sources/` page was created from it, OR
- Its content was folded into an existing wiki page, OR
- It satisfies an already-existing `[[{{RAW}}/<file>]]` wikilink in the wiki, OR
- It is a fragment processed via the fragments workflow.

**Read-but-skipped files stay where they were.** After a batch, moved-file count must equal ingested-file count. If unsure, default to *not* moving and surface the question.

## Hybrid ingest

Default: process silently in one pass. **Pause to ask** only on:

- A new top-level concept that could reshape future organization.
- Contradiction with an existing wiki claim.
- Ambiguous entity reference (two distinct things with the same name).
- Stance-heavy source where emphasis materially affects the summary.

Steps:

1. Read the source (from `{{RAW}}/inbox/` or wherever I point — any folder).
2. (Optional pause — one concise question.)
3. Write `{{WIKI}}/sources/{slug}.md` in the source's language. Set `sources: ["[[{{RAW}}/<file>]]"]` to the post-move location.
4. Create or update relevant entity/concept pages. A single source typically touches 3–15 pages.
5. Update `{{WIKI}}/index.md`.
6. Append entry to `{{WIKI}}/log.md`.
7. **Move the source to `{{RAW}}/`** — only if ingested per the move policy.

If any step fails, abort before the move. Never leave the vault half-ingested.

## Fragment ingest

Fragments are short captures (sentence, photo of a book page, screenshot + caption) dropped into `{{RAW}}/inbox/`.

Differences from hybrid:

- **Fold, don't stub.** A fragment typically extends an existing concept/entity/synthesis page as a new bullet with provenance — not a new `sources/` page. New page only when the fragment opens a new topic.
- **Batch-oriented.** One consolidated log entry, not one per fragment.

**Wikilink routing tags** — two special tags only: if the fragment body contains `[[someday]]`, create/update a `{{WIKI}}/someday/` page instead of folding. If it contains `[[ideas]]`, create/update a `{{WIKI}}/ideas/` page instead of folding. All other wikilinks follow normal ingest rules.

**Ideas page structure** — when creating or updating a `{{WIKI}}/ideas/` page:

- **What it is** — one paragraph, plain language.
- **Why it appeals** — what yearning / problem / curiosity it speaks to. Often links back to `{{WIKI}}/syntheses/` or `{{WIKI}}/concepts/`.
- **Open questions** — what I don't know yet. Bullets.
- **Notes** — accumulated bullets as the idea develops. Append-only.

**Image handling**: read the image directly — OCR text, identify subjects, pull quotes. Embed from the wiki page using `![[<filename>]]` pointing to the post-move location.

Steps, per fragment (or pair):

1. Read. For images, extract text and visual content. For pairs, combine caption with image contents.
2. Decide: extends existing page, or new topic?
   - **Extends**: append a distilled-claim bullet with provenance `[[{{RAW}}/fragments/<timestamp>]]`. Update the page's `sources` frontmatter.
   - **New topic**: create the appropriate wiki page (no `{{WIKI}}/sources/` page for the fragment itself; it lives in `{{RAW}}/fragments/` and is cited directly).
3. Move out of `{{RAW}}/inbox/` to `{{RAW}}/fragments/`. Preserve the timestamped filename — it's the fragment's identity.
   - **Untitled exception**: `untitled.md` / `untitled 1.md` (no timestamp prefix) → rename to a descriptive kebab-case slug derived from content during the move (Language rule applies).

After all fragments:

4. Update `{{WIKI}}/index.md` with any new pages.
5. Append **one consolidated entry** to `{{WIKI}}/log.md`: fragment count, pages touched, new pages.

**Pause thresholds** are higher than hybrid — only pause on the same triggers (contradiction, ambiguous entity, new top-level concept). One-sentence fragments rarely warrant a pause.

If any step fails for one fragment, abort before moving that fragment. Partial batches are fine.
