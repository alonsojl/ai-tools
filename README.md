# ai-tools

A personal [Claude Code](https://claude.com/claude-code) plugin marketplace providing agents and skills for Go development and git workflows.

## Installation

Add the marketplace (from a local checkout):

```bash
/plugin marketplace add ./ai-tools
```

Validate the marketplace definition:

```bash
claude plugin validate ./ai-tools
```

Then install any of the plugins below:

```bash
/plugin install <plugin-name>@ai-tools
```

## Plugins

### golang-engineering

Agents and skills for Go software engineering: code review, testing, concurrency, and language conventions.

- **`go-builder`** agent — implements Go code changes (writes/edits code, runs `go build`/`go vet`/`go test`/lint), then commits the result following Conventional Commits. Runs in an isolated git worktree (`isolation: worktree`) so the main working tree stays untouched. Preloads the `git-commit` and `golang-code-style` skills.

### git-tools

Skills for git workflows: conventional commits and related tooling.

- **`git-commit`** skill — analyzes the diff (staged or working tree), determines the appropriate Conventional Commits type/scope/description, stages files as needed, and creates the commit. Includes a Git Safety Protocol (no `--no-verify`, no force-push to main, no destructive operations without explicit request).

### cc-skills-golang

External plugin from [samber/cc-skills-golang](https://github.com/samber/cc-skills-golang) — referenced directly via a GitHub source (not vendored), so it tracks upstream commits automatically.

Provides 40+ Go-focused skills (code style, naming, concurrency, testing, error handling, design patterns, popular libraries, and more). `go-builder` preloads the `golang-code-style` skill from this plugin.

## Plugin dependencies

`go-builder` (in `golang-engineering`) preloads skills from the other two plugins via its `skills:` frontmatter. For those preloads to resolve, install/enable `git-tools` and `cc-skills-golang` alongside `golang-engineering`:

```bash
/plugin install golang-engineering@ai-tools
/plugin install git-tools@ai-tools
/plugin install cc-skills-golang@ai-tools
```
