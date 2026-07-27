# skills

A centralized monorepo of skills, tools, and configurations for use with Claude Code and other agent harnesses.

This repo collects independent skill modules under [`skills/`](./skills/). Each module is self-contained, with its own `SKILL.md`, license (when derived from upstream work), and documentation.

## Install all skills

### Option 1 — `skills` CLI (npm)

```bash
npx skills add ycdesu/skills
```

### Option 2 — `gh skill` (GitHub CLI, in preview)

```bash
gh skill install ycdesu/skills
```

### Option 3 — manual

```bash
git clone https://github.com/ycdesu/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/* ~/.claude/skills/
```

## Available skills

More skills will be added here over time.

### [`personal-wiki-init`](./skills/personal-wiki-init/)

> Dump fragments anywhere — half-thoughts, links, scribbles. Claude threads them into your wiki and wires the wiki links. No dashboards. No config. Stay in flow.

Bootstrap an LLM-maintained personal wiki in any project. Creates a `wiki/` folder skeleton, installs companion skills (`wiki-ingest`, `wiki-query`, `review`) under `.claude/skills/`, and appends a marker-wrapped conventions block to `CLAUDE.md` so casual edits follow the same rules. Idempotent — safe to re-run for updates.

Inspired by [Andrej Karpathy's personal wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

> **Prerequisite:** Enable the [Obsidian CLI](https://obsidian.md/help/cli) before using this skill. The wiki skills use the Obsidian CLI exclusively to open and manage vault files.

#### Option 1 — `skills` CLI (npm)

```bash
npx skills add ycdesu/skills --skill personal-wiki-init
```

#### Option 2 — `gh skill` (GitHub CLI, in preview)

```bash
gh skill install ycdesu/skills personal-wiki-init
```

#### Option 3 — manual

```bash
git clone https://github.com/ycdesu/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/personal-wiki-init ~/.claude/skills/
```

### [`vendorize-skill`](./skills/vendorize-skill/)

Add or sync skills vendored from upstream GitHub repositories. Each vendored skill folder gets a `vendored.md` manifest that records the source URL, branch, subpath, pinned commit SHA, and sync date — so future updates know exactly what to diff and reconcile.

#### Option 1 — `skills` CLI (npm)

```bash
npx skills add ycdesu/skills --skill vendorize-skill
```

#### Option 2 — `gh skill` (GitHub CLI, in preview)

```bash
gh skill install ycdesu/skills vendorize-skill
```

#### Option 3 — manual

```bash
git clone https://github.com/ycdesu/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/vendorize-skill ~/.claude/skills/
```

## Repo layout

```
skills/
├── LICENSE
├── README.md
└── skills/
    ├── personal-wiki-init/
    │   ├── SKILL.md
    │   └── templates/
    └── vendorize-skill/
        └── SKILL.md
```

## License

This repository is MIT licensed — see [LICENSE](LICENSE). Individual skill modules may carry their own upstream licenses; check each module's directory for details.
