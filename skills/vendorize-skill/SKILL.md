---
name: vendorize-skill
description: Add or update a vendored skill from an upstream GitHub repository. Use when the user asks to add a new skill from an upstream repo, sync an existing vendored skill to a newer commit, or update pinned_sha/last_synced in a vendored skill's manifest.
---

# Vendorize Skill

Manage skills vendored from upstream repos. Each vendored skill folder lives under a vendored root (referred to as `<vendored_root>` below) and contains its own `vendored.md` manifest as a sibling to the skill's other files. There is no central index — `vendored.md` is the per-skill source of truth.

## Before starting: locate `<vendored_root>`

Different repos place their skills in different locations. Resolve the vendored root once at the start of the workflow:

1. Try to discover it from existing layout:
   ```bash
   find . -name vendored.md -path '*/vendored/*' 2>/dev/null | head -1
   ```
   If a result is returned, the vendored root is the parent directory of `<name>/vendored.md` — e.g. a hit at `./agents/skills/vendored/foo/vendored.md` means `<vendored_root>` is `agents/skills/vendored`.
2. If nothing is found, prompt the user. Common conventions:
   - `.claude/skills/vendored/` (Claude Code project-level)
   - `~/.claude/skills/vendored/` (Claude Code user-level)
   - `skills/vendored/` (repo-root convention)
   - `agents/skills/vendored/` (monorepo convention)

   Ask: *"I couldn't auto-detect a vendored skills directory. Where should I put vendored skills? (default: `.claude/skills/vendored/`)"*
3. Substitute the resolved path for `<vendored_root>` in every command below.

## The per-skill manifest

Each vendored skill folder contains a `vendored.md` file with these fields:

| Field | Meaning |
|-------|---------|
| `url` | Upstream git repo URL |
| `ref` | Branch or tag to track (e.g. `main`) |
| `subpath` | Directory inside the upstream repo whose **contents** we vendor. Use `.` to vendor the repo root. |
| `pinned_sha` | Full commit SHA at last sync (`git rev-parse HEAD`) |
| `last_synced` | Date of last sync (`YYYY-MM-DD`) |
| `notes` | Record local edits here so the next sync knows to reconcile instead of overwrite |

Format:

```markdown
# Vendored

This skill folder is vendored from an upstream repo. Sync workflow lives in the `vendorize-skill` skill.

- url: https://github.com/owner/repo
- ref: main
- subpath: skills
- pinned_sha: <full sha>
- last_synced: YYYY-MM-DD
- notes:
```

To list all vendored skills: `find <vendored_root> -name vendored.md`.

## Flattening convention

Only the contents of `<subpath>` are copied — the subpath directory itself is dropped. For example, if `subpath` is `skills`, then `upstream/skills/foo/SKILL.md` lands at `<vendored_root>/<name>/foo/SKILL.md`. When diffing during a sync, always compare against `<subpath>` in the upstream clone, not the repo root.

## Adding a new vendored skill

```bash
# 1. Clone upstream at the tip of <ref>
git clone --depth=1 --branch <ref> <url> /tmp/<name>

# 2. Record the exact commit SHA
git -C /tmp/<name> rev-parse HEAD

# 3. Copy the subpath contents (flattened) into a new skill folder
mkdir -p <vendored_root>/<name>
cp -R /tmp/<name>/<subpath>/. <vendored_root>/<name>/

# 4. Clean up
rm -rf /tmp/<name>
```

Then create `<vendored_root>/<name>/vendored.md` with the fields above and commit everything together.

If `subpath` is `.` (repo root), the `cp` command works the same way.

## Syncing an existing vendored skill

```bash
# 1. Read the current manifest
cat <vendored_root>/<name>/vendored.md

# 2. Clone upstream
git clone --depth=1 --branch <ref> <url> /tmp/<name>-new

# 3. Review what changed (subpath only, and ignore the local vendored.md)
diff -ru --exclude=vendored.md <vendored_root>/<name> /tmp/<name>-new/<subpath>

# 4. Selectively apply upstream changes
#    Copy files you want to accept; skip or reconcile files with local edits
cp -R /tmp/<name>-new/<subpath>/. <vendored_root>/<name>/

# 5. Record new SHA
git -C /tmp/<name>-new rev-parse HEAD

# 6. Clean up
rm -rf /tmp/<name>-new
```

Then update `pinned_sha` and `last_synced` in `<vendored_root>/<name>/vendored.md` and commit.

Note: never let the upstream copy overwrite the local `vendored.md` — it's a downstream-side file and has no upstream counterpart.

## Excluding LLM context files

When copying from upstream, skip any LLM context files — they belong to the upstream repo's own tooling, not to this repo:

```bash
# After cp -R, remove LLM context files that were pulled in
find <vendored_root>/<name> -type f \( \
  -iname "CLAUDE.md" \
  -o -iname "CONTEXT.md" \
  -o -iname "AGENTS.md" \
\) -delete
```

Add these to the checklist as part of every add or sync.

## Handling local edits

If you modify any file inside `<vendored_root>/<name>/`, record that in the `notes` field of `vendored.md` — e.g. `notes: Extended defuddle examples`. This signals to the next sync to diff carefully rather than bulk-overwrite.

## Checklist

- [ ] `<vendored_root>` resolved (auto-detected or confirmed with user)
- [ ] `<vendored_root>/<name>/` created (add) or updated (sync)
- [ ] `<vendored_root>/<name>/vendored.md` exists with all fields filled in
- [ ] `pinned_sha` matches `git rev-parse HEAD` of the upstream clone
- [ ] `last_synced` set to today's date (`YYYY-MM-DD`)
- [ ] `notes` updated if local edits exist
- [ ] LLM context files (`CLAUDE.md`, `.cursorrules`, `AGENTS.md`, etc.) removed from vendored folder
- [ ] Local `vendored.md` not overwritten by sync
- [ ] `/tmp/<name>` or `/tmp/<name>-new` removed
- [ ] All changes committed in a single commit
