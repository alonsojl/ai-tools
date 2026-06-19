# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo hosts `marketplace`, a personal Claude Code plugin marketplace (`.claude-plugin/marketplace.json`). It catalogs plugins that bundle agents and skills for distribution via `/plugin marketplace add` and `/plugin install`.

## Structure

```
marketplace/
├── .claude-plugin/marketplace.json   # catalog of all plugins in this marketplace
└── plugins/
    ├── golang-engineering/           # Go-specific agents/skills (local source)
    │   ├── .claude-plugin/plugin.json
    │   ├── agents/go-builder.md
    │   └── skills/                   # currently empty
    └── git-tools/                    # general-purpose git workflow skills (local source)
        ├── .claude-plugin/plugin.json
        └── skills/
            ├── git-commit/SKILL.md
            └── git-flow/
                ├── SKILL.md          # team Gitflow release preparation (in Spanish)
                └── references/
                    ├── hotfix.md
                    ├── rollback.md
                    └── examples.md
```

A third catalog entry, `cc-skills-golang`, is **not vendored** — it points to the external repo `samber/cc-skills-golang` via a GitHub source (`{"source": "github", "repo": "samber/cc-skills-golang"}`), with no pinned `version`, so every new upstream commit is treated as an update.

## Working with marketplace.json

Each entry in `plugins[]` needs `name` and `source`. Two source styles are used here:

- **Local plugin** (`golang-engineering`, `git-tools`): `"source": "./plugins/<name>"`, relative to the marketplace root (the directory containing `.claude-plugin/`), and pinned with an explicit `"version"` that must be bumped on every release for users to pick up changes.
- **External GitHub plugin** (`cc-skills-golang`): `"source": {"source": "github", "repo": "owner/repo"}`. Omitting `version` means the git commit SHA is the version, so upstream commits auto-propagate as updates.

When adding a new local plugin, create `plugins/<name>/.claude-plugin/plugin.json` (requires at least `name`; this repo's convention also sets `description`, `version`, `author`) plus an `agents/` and/or `skills/<skill-name>/SKILL.md` directory, then add a matching entry to `marketplace.json`.

## Vendoring vs. linking external skills

When pulling in a skill from another repo, choose based on what it is:

- **Standalone skill file** (e.g. `git-tools/skills/git-commit/SKILL.md`, copied from `github/awesome-copilot`): vendor it — copy the `SKILL.md` (and any `references/`) verbatim into `plugins/<plugin>/skills/<skill-name>/`. Simple, robust, no upstream dependency.
- **Skill that's part of a maintained multi-skill plugin with cross-references** (e.g. `cc-skills-golang`, which has 40+ interlinked skills like `golang-naming`, `golang-lint`, `golang-structs-interfaces`): reference the whole upstream plugin via a `github` source in `marketplace.json` rather than vendoring a single skill — vendoring just one loses the cross-referenced skills and upstream improvements.

## Agent design conventions

- **Separate build vs. review roles**: `golang-engineering` follows a pattern of a read-write "builder" agent (`go-builder`) that implements + commits, kept distinct from a (planned, currently empty) read-only "reviewer" agent — so review isn't biased by the agent that wrote the code, and the reviewer can be reused on any diff.
- **`skills:` frontmatter preloads content**: listing a skill under an agent's `skills:` field injects that `SKILL.md`'s full content into the agent's context at startup (e.g. `go-builder` preloads `git-commit` and `golang-code-style`). This is independent of which plugin defines the skill — it searches all skills available in the session — but the providing plugin (e.g. `git-tools`, `cc-skills-golang`) must be installed/enabled alongside `golang-engineering` for the preload to resolve.
- **`model: opus`** (alias, not a pinned version like `claude-opus-4-8`) is used so the agent automatically tracks the latest Opus release.
- **`isolation: worktree`** is used for agents that commit (like `go-builder`): the agent runs in a separate git worktree/branch, leaving the main working tree untouched; the worktree is cleaned up automatically if no changes are made, otherwise its path/branch are returned for review and merge.

## Testing changes to this marketplace

```bash
claude plugin validate ./marketplace
/plugin marketplace add ./marketplace
/plugin install <plugin-name>@marketplace
```
