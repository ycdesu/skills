# Vault Operating Manual

The active LLM-owned area is `{{WIKI}}/` — an LLM-maintained personal wiki optimized for low-friction capture, query-first use, and active rot-prevention.

## Ownership & boundaries

- **LLM owns `{{WIKI}}/**`** — create, edit, delete pages freely to keep the wiki coherent.
- **`{{RAW}}/**` is read-only for content** — never edit source documents. You *may* relocate files within `{{RAW}}/` per the move policy below.
- **All other folders are readable** — *MUST* ask before modifying any file in them.
- **Obsidian CLI strictly required** — any vault file operation **must** use the `obsidian` CLI. Never use `mv` or other shell tools, which silently break wikilinks. **Exception: hidden files and folders** (names starting with `.`) — use standard shell tools for these; the Obsidian CLI does not handle them.

## Git commits

After completing a logical unit of work that touches vault files, create a local git commit before ending the turn so I can revert.

- **MUST NEVER** commit secrets (API keys, tokens, credentials, `.env` files, private keys, etc.).
- Group related edits into one commit; don't commit per file.
- Concise message: `ingest: <title>`, `fragments: batch of N`, `synthesis: update <slug>`, `cleanup: <what>`.
- Local only — never push unless I ask.

## Wiki folder structure

Subfolders created on-demand. Don't create empty folders.

- `{{WIKI}}/sources/` — one page per ingested source. Summary, key takeaways, outbound links.
- `{{WIKI}}/entities/` — people, organizations, products, places, tools.
- `{{WIKI}}/concepts/` — frameworks, themes, patterns, named ideas-from-sources.
- `{{WIKI}}/ideas/` — *my own* emerging ideas (brainstorms, may not happen). One page per idea.
- `{{WIKI}}/someday/` — things I *should* do but parked deliberately (commitments, not brainstorms). One page per item.
- `{{WIKI}}/syntheses/` — cross-cutting analyses, comparisons, evolving theses, filed-back query results.

Three top-level files you own:

- `{{WIKI}}/now.md` — single-glance focus page. Three sections: this quarter's goal, current milestone (with target date), this week (≤3 bullets). Pinned at the top of `index.md` via `![[now]]` — never edit the `## Now` block in `index.md` directly. **Mutable**: small factual updates (scoreboard, cadence counters, ticking off this-week bullets, bumping `updated:`) can happen freely as side effects of ingest or check-ins. Use the `review` skill's **Now** route for structural changes (quarter goal, milestone, target date, this-week rewrite).
- `{{WIKI}}/index.md` — content catalog. Updated on every ingest. Read first on every query. Links use the pipe-alias form `[[path/slug|slug]]` — never bare `[[path/slug]]`. Top-level sections in this order: `## Now`, `## Ideas`, `## Someday`, `## Sources`, `## Concepts`, `## Syntheses`, `## Entities`. Sources/Concepts/Entities are subdivided by domain subheadings (`### *Investing*`, etc.); add a new `###` subheading if none fits. All `###` subheadings use italic text (`### *Text*`).
- `{{WIKI}}/log.md` — append-only chronological log. Format and `{op}` vocabulary are in the file's header.

## Page conventions

**Frontmatter** (YAML) on every page:

```yaml
---
type: source | entity | concept | idea | someday | synthesis
tags: [tag1, tag2]
sources: ["[[{{RAW}}/slug]]"]   # or ["[[sources/slug]]"] — see below
lang: en | zh | ...
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: open | done | dropped   # idea & someday pages — done/dropped pages are unlisted from index.md
---
```

`sources` semantics by page type:
- **`source` pages** — one entry to the raw note at its post-move location (`[[{{RAW}}/<file>]]`, never `{{RAW}}/inbox/...`).
- **`entity` / `concept` / `synthesis` pages** — one or more entries to wiki source pages (`[[sources/slug]]`) substantiating claims.
- **`idea` / `someday` pages** — optional. Empty `sources` is fine for items from my own thinking.

**Body**:

- One H1 matching the filename (exception: `now.md` — no H1).
- Obsidian `[[wikilinks]]` for all internal references — never plain markdown links.
- Obsidian embed for images: `![[image.png]]` — never plain markdown image syntax.
- Concise and scannable. Bullets > paragraphs.

**Language rule** — match the source's language for summary, H1, body, **and filename**. English source → kebab-case (`attention-economy.md`). Chinese source → Chinese filename (`注意力經濟.md`). Mixed-language wikis are fine. Deviate only when I explicitly ask.

**Voice rule** — wiki pages are my personal notes. Always first-person: `I` / `me` / `my` (English) or `我` / `我的` (Chinese). Never `User` / `the user` / `使用者`, never write *about* me in third person.

- ❌ "User clarified that the deferral isn't…" → ✅ "I clarified that the deferral isn't…"

Exceptions: source pages summarizing an article that uses "the user" generically (preserve the source's framing); section headers explicitly attributed (e.g., `## Observations (Claude)`) — but inside that prose still use `I` for me, name `Claude` directly.

**Filenames**: descriptive, not cryptic (`attention-economy.md`, not `ae.md`).

**Wikilink resolution** — how to treat `[[x]]` when ingesting:

- **Target missing**: do *not* auto-create on first reference. Materialize only once ≥2 notes reference it. Before that, leave it unresolved. (Exception: `[[ideas]]` and `[[someday]]` routing tags in fragments always create immediately — see `wiki-ingest` skill.)
- **Target exists but empty**: intentional aggregator node. Don't auto-populate; rely on backlinks. Populate only when I ask.
- **Target exists with content**: fold new material in, update `sources:` and `updated:`, pause on contradictions.

## Workflows

All workflows are defined as skills:
- **Ingest** (hybrid, fragments, move policy) → `wiki-ingest` skill
- **Query** (read index, cite sources, file-back offer) → `wiki-query` skill
- **Review** (Now / Someday / Lint / Tour) → `review` skill

## Interaction style

- Silent ingest by default. Surface pauses concisely — one question at a time.
- Short status updates as you go ("wrote `sources/x`, updating entity pages").
- When filing-back from a discussion, confirm the target filename before writing.
- Cite wikilinks when answering. Never fabricate page contents.
