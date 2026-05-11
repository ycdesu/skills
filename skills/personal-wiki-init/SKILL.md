---
name: personal-wiki-init
description: Bootstrap a personal-wiki project. Creates the wiki/ folder skeleton, installs the wiki-ingest / wiki-query / review skills, and appends a marked conventions block to CLAUDE.md. Run once per project; safe to re-run for updates.
---

# personal-wiki-init

Scaffold an opinionated, LLM-maintained personal wiki in the consumer's project. After this runs, the project has:

- A `wiki/` folder with the canonical subfolders and three top-level files.
- `.claude/skills/{wiki-ingest, wiki-query, review}/` populated.
- A marker-wrapped conventions block in `CLAUDE.md` so casual edits (no skill invocation) follow the right rules.

The skill is **idempotent** — safe to re-run. New consumer content (e.g., `now.md`, `index.md`) is never clobbered. Skill files use diff-and-ask. The CLAUDE.md block is replaced in place between markers.

## Template substitution

Every file under `templates/` is a template. Substitute these tokens before writing each file:

| Token | Replace with |
|---|---|
| `{{WIKI}}` | Chosen wiki root, no trailing slash. Default: `wiki` |
| `{{RAW}}` | Chosen raw root, no trailing slash. Default: `raw` |
| `{{TODAY}}` | Today's date, `YYYY-MM-DD` |

## Step-by-step

### 1. Confirm wiki and raw roots

Ask one at a time, accept defaults:

- Wiki path (default: `wiki/`)
- Raw notes path (default: `raw/`)

Record as `<wiki>` and `<raw>`.

### 2. Create folder skeleton (idempotent)

Create these folders if missing. If a folder already exists, leave it alone.

- `<wiki>/`
- `<wiki>/sources/`
- `<wiki>/entities/`
- `<wiki>/concepts/`
- `<wiki>/ideas/`
- `<wiki>/someday/`
- `<wiki>/syntheses/`
- `<raw>/`
- `<raw>/inbox/`

### 3. Drop wiki starter files

For each file under this skill's `templates/wiki/`, target `<wiki>/<filename>`:

- **If target exists**: skip and report. These accumulate user content fast — never overwrite.
- **If target absent**: write the template with tokens substituted.

### 4. Install workflow skills

For each file under this skill's `templates/skills/`, target `.claude/skills/<same-relative-path>` (creating parent dirs as needed). Substitute tokens before any comparison or write.

- **If target absent**: write; report as installed.
- **If target matches the substituted template**: skip silently.
- **If target exists and differs** — ask: *"Existing `<path>` differs from template. Keep / overwrite / show diff?"*
  - **Keep** → skip; report as kept.
  - **Overwrite** → replace; report as overwritten.
  - **Show diff** → display unified diff, then re-ask keep/overwrite.

### 5. Update CLAUDE.md

Source: this skill's `templates/CLAUDE-conventions.md`.

Target: `CLAUDE.md` at the project root (where the consumer is running Claude Code).

Substitute tokens (`{{WIKI}}`, `{{RAW}}`) in the template before wrapping it in markers. The block format:

```
<!-- personal-wiki:start -->
{contents of templates/CLAUDE-conventions.md, with tokens substituted}
<!-- personal-wiki:end -->
```

Three cases:

- **`CLAUDE.md` absent**: create it with just the marker-wrapped block.
- **Present, markers found**: replace everything between `<!-- personal-wiki:start -->` and `<!-- personal-wiki:end -->` (markers themselves stay). Do not touch any content outside the markers.
- **Present, markers absent**: append the marker-wrapped block to the end of the file (preceded by a blank line if the file doesn't end with one).

### 6. Append init log entry

Append to `<wiki>/log.md`:

```
## [<today>] init | personal-wiki bootstrap
- Wiki root: <wiki>/
- Raw root: <raw>/
- Installed skills: wiki-ingest, wiki-query, review
```

Replace `<today>` with the current `YYYY-MM-DD`. The log is append-only — re-runs add a new entry.

### 7. Report file actions

Print a grouped summary of every file touched in this run. Group by category:

```
Folders created: <list>  (or "none — all already existed")
Wiki starter files: <list of created> · skipped (exists): <list>
Skill files: <list of installed> · kept: <list> · overwritten: <list>
CLAUDE.md: created | block-updated | block-appended
log.md: entry appended
```

Keep it scannable. One line per category.

### 8. Post-install summary

Print this verbatim, substituting paths if the consumer chose non-default `<wiki>` / `<raw>`:

```
Installed skills (in .claude/skills/):
- /wiki-ingest — ingest sources or inbox fragments. Run when adding raw notes.
    Usage: drop a file into <raw>/inbox/, then say "ingest" or "ingest fragments".
- /wiki-query — ask questions about your wiki contents. No trigger needed —
    just ask anything about your vault.
- /review — review the wiki. Routes to Now (default), Someday, Lint, or Tour.
    Usage: say "review", "lint", "tour", or "someday review".

Next steps:
  1. Open CLAUDE.md and skim the conventions block (between
     <!-- personal-wiki:start --> markers) so you know how I'll handle your wiki.
  2. Edit <wiki>/now.md — fill in your quarter goal, milestone, and this week.
  3. Drop your first source into <raw>/inbox/ and say "ingest".
  4. Install the Obsidian CLI (`obsidian-cli`) if you use Obsidian, so wiki
     files can be opened directly from the terminal:
     https://github.com/Vinzent03/obsidian-cli

Re-run /personal-wiki-init anytime to update conventions or skills.
```

## Operating notes

- **Read-only on existing wiki content.** Never edit files inside `<wiki>/` other than the three starter files (and only when they're absent), and `log.md` for the init entry append.
- **Never run this skill against a vault that already has live content unless the consumer asks.** Re-runs in a populated vault are safe by design (idempotent + diff-and-ask), but the consumer should know it's about to happen.
- **Don't fabricate paths.** If `<wiki>` or `<raw>` already contain a different layout from what this skill expects, surface the conflict and ask before creating folders.
