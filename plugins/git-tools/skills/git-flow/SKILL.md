---
name: git-flow
description: Team Gitflow workflow for releases and hotfixes.
allowed-tools: Bash
---

# Gitflow — Release & Hotfix

## Overview

Prepares branches to release tickets to development, sandbox, and production following the team's Gitflow workflow. All merges use `--no-ff`.

## Branch Conventions

| Branch | Purpose |
|--------|---------|
| `master` | Production |
| `develop` | Integration |
| `feature_ts<NUM>` | Per ticket (multi-ticket: `feature_ts<NUM1>_ts<NUM2>`) |
| `develop_ts<NUM>` | QA per ticket — deployed to sandbox after feature push |
| `release_<X.Y.Z>` | Release |
| `hotfix_<X.Y.Z>` | Hotfix — branches from `develop`, named after current tag (version with the issue) |

## Auto-detection

When the user says "prepare this branch" (or similar), detect the current branch type:

```bash
git branch --show-current
```

| Current branch | Detected type |
|----------------|---------------|
| `develop_ts<NUM>` | develop |
| `feature_ts<NUM>` | feature |
| `hotfix_<X.Y.Z>` | hotfix |

Ask the user with selectable options **where to deploy**:

| Option | Target | Steps |
|--------|--------|-------|
| `[Sandbox]` | Feature → push (QA validates, deployed to sandbox) | develop_ts: 1–3 / feature_ts: 2–3 |
| `[Production]` | All the way to master + tag | develop_ts: 1–7 / feature_ts: 2–7 |

**If `hotfix`:**
Don't ask — always goes to production. Read `references/hotfix.md` and execute from H3.

**After selection, execute all steps without asking.** Only stop on merge conflicts.

## Workflow

### 1. Create feature from develop

```bash
git checkout develop
git pull origin develop
git checkout -b feature_ts<NUM>
```

Multi-ticket: use `feature_ts<NUM1>_ts<NUM2>` in the order specified by the user.

### 2. Merge develop_ts into feature

```bash
git merge --no-ff develop_ts<NUM>
```

Multi-ticket: merge each corresponding `develop_ts<NUM>`.

### 3. Push feature (deploy to sandbox)

```bash
git push origin feature_ts<NUM>
```

If target is **sandbox**, the flow ends here.

### 4. Merge feature(s) into develop

```bash
git checkout develop
git merge --no-ff feature_ts<NUM>
```

Don't push `develop` yet.

### 5. Create release

Get the latest tag and calculate the next patch automatically:

```bash
git tag -l --sort=-v:refname | grep -E '^[0-9]+\.[0-9]+\.[0-9]+$' | head -1
git checkout -b release_<X.Y.Z>
```

Use **patch** by default (`1.0.0` → `1.0.1`).

### 6. Merge release into master

```bash
git checkout master
git pull origin master
git merge --no-ff release_<X.Y.Z>
```

### 7. Tag and push

```bash
git tag -a <X.Y.Z> -m "Merge branch 'release_<X.Y.Z>'"
git push origin develop master <X.Y.Z>
git checkout develop
```

All three references (`develop`, `master`, tag) are pushed in a single command. Leave the user on `develop` when done.

## Merge Conflicts

Don't abort without notifying. Show `git status`, offer options: resolve manually / abort / leave in conflict. After resolving: `git add` + `git commit` (no `-m`, keep default merge message). Don't proceed to the next step until the merge is clean.

## Additional Flows

Read the corresponding file only when needed:

- **Hotfix**: `references/hotfix.md`

## Final Checklist

**Sandbox:**
- [ ] `develop_ts<NUM>` merged into feature
- [ ] Feature pushed

**Production:**
- [ ] `develop` has all features for the release
- [ ] `release_<X.Y.Z>` exists
- [ ] `master` has the release merge
- [ ] Annotated tag (`-a`) with message `"Merge branch 'release_<X.Y.Z>'"`
- [ ] Push of `develop`, `master`, and tag together
- [ ] User on `develop`
