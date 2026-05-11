# skills

A centralized monorepo of skills, tools, and configurations for use with Claude Code and other agent harnesses.

Each skill under [`skills/`](./skills/) is self-contained, with its own `SKILL.md`, license (when derived from upstream work), and documentation.

## Contents

- [Installation](#installation)
  - [Install all skills](#install-all-skills)
  - [Install a single skill](#install-a-single-skill)
- [Available skills](#available-skills)
- [Repo layout](#repo-layout)
- [License](#license)

## Installation

Three methods are supported. Pick one — they install the same files.

| Method | When to use |
| --- | --- |
| `skills` CLI (npm) | Simplest. Requires Node.js. |
| `gh skill` (GitHub CLI, preview) | If you already use `gh`. |
| Manual `git clone` | No extra tooling needed. |

### Install all skills

```bash
# npm
npx skills add ycdesu/skills

# gh CLI
gh skill install ycdesu/skills

# manual
git clone https://github.com/ycdesu/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/* ~/.claude/skills/
```

### Install a single skill

Replace `<skill>` with the skill folder name (e.g. `native-git-stack`).

```bash
# npm
npx skills add ycdesu/skills --skill <skill>

# gh CLI
gh skill install ycdesu/skills <skill>

# manual
git clone https://github.com/ycdesu/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/<skill> ~/.claude/skills/
```

## Available skills

| Skill | Description |
| --- | --- |
| [`native-git-stack`](./skills/native-git-stack/) | Manage stacked branches and PRs with only `git` and `gh`. |
| [`personal-wiki-init`](./skills/personal-wiki-init/) | Bootstrap an LLM-maintained personal wiki in any project. |
| [`vendorize-skill`](./skills/vendorize-skill/) | Vendor and sync skills from upstream GitHub repositories. |

More skills will be added over time.

### `native-git-stack`

Manage stacked branches and pull requests using only native `git` and the GitHub CLI (`gh`) — no extensions or GitHub Stacked PRs feature required. Reorganizes your code into separate, atomic commits and branches so each change can be reviewed independently.

Derived from GitHub's [`gh-stack`](https://github.com/github/gh-stack) skill.

### `personal-wiki-init`

Bootstrap an LLM-maintained personal wiki in any project. Creates a `wiki/` folder skeleton, installs companion skills (`wiki-ingest`, `wiki-query`, `review`) under `.claude/skills/`, and appends a marker-wrapped conventions block to `CLAUDE.md` so casual edits follow the same rules. Idempotent — safe to re-run for updates.

Inspired by [Andrej Karpathy's personal wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

> **Prerequisite:** Enable the [Obsidian CLI](https://obsidian.md/help/cli) before using this skill. The wiki skills use the Obsidian CLI exclusively to open and manage vault files.

### `vendorize-skill`

Add or sync skills vendored from upstream GitHub repositories. Each vendored skill folder gets a `vendored.md` manifest that records the source URL, branch, subpath, pinned commit SHA, and sync date — so future updates know exactly what to diff and reconcile.

## Repo layout

```
skills/
├── LICENSE
├── README.md
└── skills/
    ├── native-git-stack/
    │   ├── LICENSE
    │   └── SKILL.md
    ├── personal-wiki-init/
    │   ├── SKILL.md
    │   └── templates/
    └── vendorize-skill/
        └── SKILL.md
```

## License

This repository is MIT licensed — see [LICENSE](LICENSE). Individual skill modules may carry their own upstream licenses; check each module's directory for details.
