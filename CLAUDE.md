# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo hosts `ai-tools`, a personal Claude Code plugin marketplace (`.claude-plugin/marketplace.json`). It catalogs plugins that bundle agents and skills for distribution via `/plugin marketplace add` and `/plugin install`.

## Structure

```
ai-tools/
├── .claude-plugin/marketplace.json
└── plugins/
    ├── golang-engineering/
    │   ├── .claude-plugin/plugin.json
    │   └── agents/golang-engineer.md
    └── git-tools/
        ├── .claude-plugin/plugin.json
        └── skills/
            ├── git-commit/SKILL.md
            └── git-flow/
                ├── SKILL.md
                └── references/hotfix.md
```

A third catalog entry, `cc-skills-golang`, is **not vendored** — it points to the external repo `samber/cc-skills-golang` via a GitHub source (`{"source": "github", "repo": "samber/cc-skills-golang"}`), with no pinned `version`, so every new upstream commit is treated as an update.

## Working with marketplace.json

Each entry in `plugins[]` needs `name` and `source`. Two source styles are used here:

- **Local plugin** (`golang-engineering`, `git-tools`): `"source": "./plugins/<name>"`, relative to the marketplace root (the directory containing `.claude-plugin/`), and pinned with an explicit `"version"` that must be bumped on every release for users to pick up changes.
- **External GitHub plugin** (`cc-skills-golang`): `"source": {"source": "github", "repo": "owner/repo"}`. Omitting `version` means the git commit SHA is the version, so upstream commits auto-propagate as updates.

When adding a new local plugin, create `plugins/<name>/.claude-plugin/plugin.json` (requires at least `name`; this repo's convention also sets `description`, `version`, `author`) plus an `agents/` and/or `skills/<skill-name>/SKILL.md` directory, then add a matching entry to `marketplace.json`.

## Vendoring vs. linking external skills

- **Standalone skill file** (e.g. `git-commit/SKILL.md`): vendor it — copy the `SKILL.md` (and any `references/`) into `plugins/<plugin>/skills/<skill-name>/`.
- **Skill that's part of a maintained multi-skill plugin with cross-references** (e.g. `cc-skills-golang`): reference the whole upstream plugin via a `github` source in `marketplace.json` rather than vendoring a single skill.

## Agent design conventions

- **Separate build vs. review roles**: `golang-engineering` follows a pattern of a read-write implementation agent (`golang-engineer`) distinct from a read-only reviewer agent.
- **`skills:` frontmatter preloads content**: listing a skill under an agent's `skills:` field injects that `SKILL.md`'s full content into the agent's context at startup. The providing plugin must be installed/enabled for the preload to resolve.
- **`model: opus`** (alias) is used so the agent automatically tracks the latest Opus release.
- **`isolation: worktree`** is used for agents that commit: the agent runs in a separate git worktree/branch, leaving the main working tree untouched.

## Testing changes to this marketplace

```bash
claude plugin validate ./ai-tools
/plugin marketplace add ./ai-tools
/plugin install <plugin-name>@ai-tools
```
