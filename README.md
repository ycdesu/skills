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

### [`native-git-stack`](./skills/native-git-stack/)

Manage stacked branches and pull requests using only native `git` and the GitHub CLI (`gh`), with no extensions or GitHub Stacked PRs feature required. Reorganizes your code into separate, atomic commits and branches so each change can be reviewed independently. Derived from GitHub's [`gh-stack`](https://github.com/github/gh-stack) skill.

#### Option 1 — `skills` CLI (npm)

```bash
npx skills add ycdesu/skills --skill native-git-stack
```

#### Option 2 — `gh skill` (GitHub CLI, in preview)

```bash
gh skill install ycdesu/skills native-git-stack
```

#### Option 3 — manual

```bash
git clone https://github.com/ycdesu/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/native-git-stack ~/.claude/skills/
```

## Repo layout

```
skills/
├── LICENSE
├── README.md
└── skills/
    └── native-git-stack/
        ├── LICENSE
        └── SKILL.md
```

## License

This repository is MIT licensed — see [LICENSE](LICENSE). Individual skill modules may carry their own upstream licenses; check each module's directory for details.
