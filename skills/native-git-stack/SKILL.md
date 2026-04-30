---
name: native-git-stack
description: >
  Stacked branches and stacked PRs with native git + gh (no extensions, no GitHub
  Stacked PRs feature). Use when the user wants to: split one branch into many
  small PRs ("split this branch", "break this up", "make smaller PRs", "slice
  into a stack"), create a chain of dependent PRs from scratch, rebase or sync a
  stack after trunk moves, restructure a stack (reorder/drop/rename layers),
  recover after a squash-merge, or edit a mid-stack layer and cascade up. Goal:
  each PR ~200-300 LOC and individually reviewable.
metadata:
  author: yc
  version: "0.1.0"
---

# native-git-stack

A **stack** is an ordered list of branches where each one builds on the one below it, rooted on a trunk branch (`main`, `develop`, etc.). Each branch maps to one PR whose base is the branch below it, so reviewers see only that layer's diff.

```
<trunk>
 └── feat/0-auth        → PR #1 (base: <trunk>)        - bottom (closest to trunk)
  └── feat/1-api        → PR #2 (base: feat/0-auth)
   └── feat/2-frontend  → PR #3 (base: feat/1-api)     - top (furthest from trunk)
```

The **bottom** is closest to trunk; the **top** is furthest. "Up" = away from trunk, "down" = toward trunk. `<trunk>` is a placeholder — substitute the repo's default branch.

This skill uses only native `git` (2.38+) and the GitHub CLI (`gh`). It works on accounts and repos that do not have GitHub's Stacked PRs feature enabled.

## Prerequisites

- **Git 2.38+** — required for `git rebase --update-refs`. Check with `git --version`.
- **GitHub CLI v2.0+**, authenticated (`gh auth status`).
- One-time configuration:
  ```bash
  git config --global rerere.enabled true            # remember conflict resolutions
  git config --global remote.pushDefault origin      # only if multiple remotes
  ```

## Agent rules

All commands must be runnable non-interactively — editors, prompts, and TUIs will hang.

1. **Name every branch with a zero-based index.** Pattern: `<type>/<index>-<description>` (e.g. `feat/0-auth`, `feat/1-api`). The index encodes stack position and must be unique within the stack. Re-index branches above any insertion point.
2. **Branch from the correct parent.** `git checkout -b <child> <parent>` — the parent is the branch immediately below in the stack, not trunk.
3. **Always set `--base` and `--head` on `gh pr create`.** Without `--base`, the PR defaults to the repo's default branch and shows the entire diff from trunk.
4. **Cascade-rebase from the top with `--update-refs`.** Run `GIT_SEQUENCE_EDITOR=: git rebase -i --update-refs <trunk>` from the topmost branch. The `GIT_SEQUENCE_EDITOR=:` prefix accepts the default todo list without opening an editor.
5. **Use `git push --force-with-lease` after a rebase.** Never plain `--force`.
6. **Stage deliberately.** `git add <specific files>` per logical concern. Avoid `git commit -am`.
7. **Push the branch before `gh pr create`.** The head branch must exist on the remote.
8. **Pass `--title` and `--body` (or `--fill`) to `gh pr create`.** Otherwise it opens an editor.
9. **Back up before destructive resets.** `git tag wip-backup` before `git reset --hard`, `git reset --soft <trunk>`, or restructuring rebases.
10. **Never rename a branch with an open PR.** GitHub does not allow editing a PR's head; the rename closes the PR.

## Stack structure

Each branch should be a **discrete, logical unit of work** reviewable on its own. Foundational changes go in lower (earlier) branches; code that depends on them goes in higher (later) branches. If code in one layer depends on code in another, the dependency must be in the same branch or a lower one.

Plan layers before writing code:

```
<trunk>
 └── feat/0-models      ← shared types, schema
  └── feat/1-api        ← API routes that use the models
   └── feat/2-frontend  ← UI that calls the APIs
    └── feat/3-tests    ← integration tests
```

**Branch naming.** Required pattern: `<type>/<index>-<description>` — e.g. `feat/0-auth-middleware`, `fix/0-login-redirect`. The index makes stack order unambiguous in `git branch` output, PR lists, and CI logs without tracing parent pointers.

**One stack, one story.** A reviewer reading the PRs in sequence should see a coherent progression. Start a new stack for unrelated work. Trivial incidental fixes (one-line typo) can ride along; if a fix grows, give it its own stack.

**When to start a new branch.** Switching layers (backend → frontend), moving from logic to tests/docs, different reviewer audience, or the current PR is already at ~200-300 LOC.

## Quick reference

| Task | Command |
|------|---------|
| Create bottom of stack | `git checkout -b feat/0-foo <trunk>` |
| Add layer on top | `git checkout -b feat/1-bar feat/0-foo` |
| Push whole stack | `git push -u origin feat/0-foo feat/1-bar feat/2-baz` |
| Create stacked PR | `gh pr create --base <parent> --head <branch> --title "..." --body "..."` |
| Sync trunk + cascade rebase | from top branch: `GIT_SEQUENCE_EDITOR=: git rebase -i --update-refs origin/<trunk>` |
| Continue / abort rebase | `git rebase --continue` / `git rebase --abort` |
| Push after rebase | `git push --force-with-lease origin <each branch>` |
| List my open PRs | `gh pr list --author "@me" --json number,title,headRefName,baseRefName,state,url` |
| Restructure stack | `git rebase -i --update-refs <trunk>` from top, edit todo list |
| Update PR base on GitHub | `gh pr edit <PR#> --base <new-parent>` |
| Tear down | `gh pr close <PR#> --delete-branch` then `git branch -D <branch>` |

---

## Workflows

### 1. Create a stack from scratch

```bash
# Bottom layer (index 0 = closest to trunk)
git checkout -b feat/0-auth <trunk>
git add internal/auth/middleware.go internal/auth/middleware_test.go
git commit -m "Add auth middleware with token validation"

# Each subsequent layer branches from the layer below, NOT from trunk
git checkout -b feat/1-api-routes feat/0-auth
git add internal/api/users.go
git commit -m "Add user API routes"

git checkout -b feat/2-frontend feat/1-api-routes
git add web/dashboard.tsx
git commit -m "Add user dashboard UI"

# Push every branch
git push -u origin feat/0-auth feat/1-api-routes feat/2-frontend

# Create PRs bottom-up, each --base set to the layer below
gh pr create --base <trunk>           --head feat/0-auth       --title "Auth middleware" --body "..."
gh pr create --base feat/0-auth       --head feat/1-api-routes --title "User API"        --body "..."
gh pr create --base feat/1-api-routes --head feat/2-frontend   --title "Dashboard UI"    --body "..."
```

Add `--draft` for draft PRs. Use `--fill` to derive title/body from commits.

### 2. Mid-stack changes

You're on `feat/2-frontend` and realize you need a new API endpoint. Don't add it on the frontend branch — that puts the change in the wrong PR. Navigate down, then cascade up.

```bash
git checkout feat/1-api-routes
git add internal/api/users.go
git commit -m "Add get-user endpoint"

# Cascade-rebase from the top so --update-refs sees every layer
git checkout feat/2-frontend
GIT_SEQUENCE_EDITOR=: git rebase -i --update-refs <trunk>

git push --force-with-lease origin feat/1-api-routes feat/2-frontend
```

`--update-refs` (Git 2.38+) inserts `update-ref` lines at branch boundaries, moving every intermediate branch pointer atomically.

### 3. Address review feedback on a mid-stack PR

```bash
gh pr checkout 42                     # checks out the PR's head branch
git add internal/auth/middleware.go
git commit -m "Address review: tighten token expiry"

git checkout <top-branch>             # e.g. feat/2-frontend
GIT_SEQUENCE_EDITOR=: git rebase -i --update-refs <trunk>

git push --force-with-lease origin feat/0-auth feat/1-api-routes feat/2-frontend
```

### 4. Sync after trunk moves

```bash
git fetch origin
git checkout <top-branch>
GIT_SEQUENCE_EDITOR=: git rebase -i --update-refs origin/<trunk>
git push --force-with-lease origin feat/0-auth feat/1-api-routes feat/2-frontend
```

If the rebase conflicts, see §6.

### 5. Squash-merge recovery

When the bottom PR is squash-merged on GitHub, its original commits no longer exist on `<trunk>` — they were replaced by one squash commit. A naive rebase would replay them as duplicates. Use `git rebase --onto` to skip them.

```bash
git fetch origin
git checkout <trunk> && git pull --ff-only

# Re-base the new bottom onto trunk, dropping the merged branch's commits
git checkout feat/1-api-routes
git rebase --onto <trunk> feat/0-auth feat/1-api-routes
#                          ^upstream  ^branch

# Update its PR base on GitHub
gh pr edit <PR#> --base <trunk>

# Cascade the rest
git checkout feat/2-frontend
GIT_SEQUENCE_EDITOR=: git rebase -i --update-refs origin/<trunk>

git push --force-with-lease origin feat/1-api-routes feat/2-frontend
git branch -D feat/0-auth
```

With `rerere` enabled, recurring conflicts auto-resolve.

### 6. Resolve rebase conflicts

```bash
git status                            # lists files with "both modified"
# resolve markers, then:
git add path/to/resolved.go
git rebase --continue
# or, to bail entirely:
git rebase --abort
```

`git config --global rerere.enabled true` records every manual resolution and replays it on recurrence — essential for long-lived stacks.

### 7. Inspect the stack

```bash
# Open PRs you authored, sorted bottom-first
gh pr list --author "@me" --state open \
  --json number,title,headRefName,baseRefName,state,url,isDraft \
  --jq 'sort_by(.headRefName)'

# Does <branch> need a rebase?
git merge-base --is-ancestor <parent> <branch> && echo "up to date" || echo "needs rebase"

# Commits unique to <branch> relative to its parent
git log --oneline <parent>..<branch>
```

### 8. Restructure a stack

#### Reorder, drop, or squash layers

`git rebase -i --update-refs` exposes the entire stack as one todo list.

```bash
git checkout <top-branch>
git rebase -i --update-refs <trunk>

# In the editor:
#   pick abc1234 Add auth middleware
#   update-ref refs/heads/feat/0-auth
#   pick def5678 Add user API routes
#   update-ref refs/heads/feat/1-api-routes
#   pick 7890abc Add user dashboard UI
#   update-ref refs/heads/feat/2-frontend
#
# Drop a branch: delete its `pick` line(s) AND its `update-ref` line.
# Reorder:       move the `pick` lines (and their `update-ref` lines).
# Squash:        change `pick` to `squash` or `fixup`.

git push --force-with-lease origin feat/0-auth feat/1-api-routes feat/2-frontend
gh pr edit <PR#> --base <new-parent>     # if any branch's parent changed
```

#### Rename a branch

Renaming a branch with an open PR effectively closes the PR — GitHub does not allow editing a PR's head.

```bash
git branch -m feat/old feat/new
git push -u origin feat/new
git push origin --delete feat/old
gh pr close <old PR#>
gh pr create --base <parent> --head feat/new --title "..." --body "..."
gh pr edit <downstream PR#> --base feat/new
```

Avoid renames on branches with open PRs.

### 9. Split one branch into a stack

Triggered by phrases like *"split this branch"*, *"break this up"*, *"slice into a stack"*. You have a single branch (`wip`) with several commits and want to slice it into small reviewable PRs. Pick the approach that matches the state of your work.

**Always back up first:**

```bash
git checkout wip
git tag wip-backup           # `git reset --hard wip-backup` restores it
```

#### Approach A — clean commits, group by commit boundary

If commits on `wip` are already atomic and just need grouping:

```bash
# wip has C1 C2 C3 C4 C5 on top of <trunk>; want feat/0=C1+C2, feat/1=C3, feat/2=C4+C5
git checkout -b feat/0-auth <trunk>
git cherry-pick <C1> <C2>

git checkout -b feat/1-api feat/0-auth
git cherry-pick <C3>

git checkout -b feat/2-frontend feat/1-api
git cherry-pick <C4> <C5>
```

Or, if commits are already in the right order on `wip`, just plant branch pointers:

```bash
git branch feat/0-auth     <sha-of-C2>
git branch feat/1-api      <sha-of-C3>
git branch feat/2-frontend <sha-of-C5>     # tip of wip
```

#### Approach B — messy commits, manual re-stage by file groups

You decide the grouping; rebuild each branch from a clean working tree:

```bash
git checkout wip
git reset --soft <trunk>             # all changes pile up in the index
git reset                            # unstage; working tree still has every diff

git checkout -b feat/0-models <trunk>
git add internal/models/ db/migrations/
git commit -m "Add user models and migration"

git checkout -b feat/1-api feat/0-models
git add internal/api/
git commit -m "Add user API routes"

git checkout -b feat/2-frontend feat/1-api
git add web/
git commit -m "Add user dashboard UI"

git status                           # must be clean
```

#### Approach C — split a single fat commit mid-history

```bash
git rebase -i <trunk>
# Change `pick` → `edit` on the commit to split. When rebase stops:
git reset HEAD~                      # uncommit, keep changes in working tree
git add <files for first concern>
git commit -m "First concern"
git add <files for second concern>
git commit -m "Second concern"
git rebase --continue
```

Then use Approach A to plant branch pointers.

#### Approach D — LLM-driven atomic re-commit (opt-in)

Use when the user wants the agent to design the layer split rather than specifying it themselves. Trigger phrases include *"let the LLM decide the grouping"*, *"split atomically"*, or *"figure out how to break this up"*.

The agent must:

1. Run `git diff <trunk>...HEAD` and read the full changeset.
2. **Propose a layer plan and wait for explicit user approval before any destructive step.** The plan must list: branch names with indexes, one-line description per layer, which files (and which hunks, if a file is split) go where.
3. After approval: `git tag wip-backup`.
4. Soft-reset and unstage:
   ```bash
   git reset --soft <trunk>
   git reset
   ```
5. Build each layer:
   ```bash
   git checkout -b feat/0-<desc> <trunk>
   git add <files for layer 0>
   git commit -m "<message>"

   git checkout -b feat/1-<desc> feat/0-<desc>
   git add <files for layer 1>
   git commit -m "<message>"
   ```
6. `git status` to verify nothing was dropped (working tree must be clean).
7. Report final branches and ask whether to push and open PRs.

The mandatory approval step keeps the user in control of the design.

#### After A / B / C / D

Push and open PRs as in §1, then remove the safety net:

```bash
git push -u origin feat/0-foo feat/1-bar feat/2-baz
gh pr create --base <trunk>    --head feat/0-foo --title "..." --body "..."
gh pr create --base feat/0-foo --head feat/1-bar --title "..." --body "..."
gh pr create --base feat/1-bar --head feat/2-baz --title "..." --body "..."

# Once reviewed:
git tag -d wip-backup
git branch -D wip
```

### 10. Tear down a stack

```bash
gh pr close <PR#1> --delete-branch
gh pr close <PR#2> --delete-branch
gh pr close <PR#3> --delete-branch

git checkout <trunk>
git branch -D feat/0-foo feat/1-bar feat/2-baz
```

Omit `--delete-branch` to keep remote branches and PRs.

---

## Exit codes

Native `git` and `gh` exit codes — no custom codes.

| Command | Non-zero means |
|---------|---------------|
| `git rebase` | Conflict or other failure. `git status`, then `git rebase --continue` (after resolving) or `git rebase --abort`. |
| `git push --force-with-lease` | Remote moved since last fetch. `git fetch` and re-rebase before pushing. |
| `gh pr create` | Auth, network, or precondition failure. Check `gh auth status`; ensure the head branch was pushed. **Non-idempotent** — running it twice creates duplicate PRs. |
| `gh pr edit` | PR doesn't exist, you don't own it, or new base is invalid. |

For scripting, check before creating: `gh pr list --head <branch> --json number --jq 'length'`.

## Limitations

1. **Linear stacks only.** Each branch has one parent. Use separate stacks for parallel workstreams.
2. **No first-class "view" command.** Use `gh pr list --author "@me" --json …` (§7).
3. **PR head branches cannot be edited.** Renaming a head closes the PR. Avoid renames on open-PR branches.
4. **Multiple remotes need explicit config.** `git config remote.pushDefault origin` or pass the remote explicitly.
5. **`gh pr create` is non-idempotent.** Running it twice creates duplicate PRs.
6. **No automatic stack UI on GitHub.** Paste a "Stack:" navigation block into each PR body (see Appendix).
7. **`git rebase --update-refs` requires Git 2.38+.** Older versions need manual pointer updates or `git rebase --onto` per layer.

## Appendix: PR-body cross-link helper

```markdown
**Stack:**
- #10 feat/0-auth (base: <trunk>)               ← bottom
- #11 feat/1-api-routes (base: feat/0-auth)
- #12 feat/2-frontend (base: feat/1-api-routes) ← top
```

Generate the block from current open PRs:

```bash
gh pr list --author "@me" --state open \
  --json number,title,headRefName,baseRefName \
  --jq 'sort_by(.headRefName) | .[] | "- #\(.number) \(.headRefName) (base: \(.baseRefName))"'
```

Then `gh pr edit <PR#> --body "$(cat body.md)"`.
