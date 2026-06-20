# Hotfix Flow

Use this flow to resolve a production incident that can't wait for the next regular release cycle. **Team variant: hotfix branches from `develop`, not `master`** (unlike classic Gitflow). Once resolved, it merges back into `develop` and follows the standard release flow from there.

## Step H0 — Pre-validation

Same validations as the main flow (correct repo, clean working tree, reachable remotes). Also confirm with the user:

- **Short description of the incident** — used in the tag message.

Get the current tag (the production version where the issue occurred):

```bash
git tag -l --sort=-v:refname | grep -E '^[0-9]+\.[0-9]+\.[0-9]+$' | head -1
```

The hotfix branch uses **the current tag** (the version with the issue). The next patch is used later for the release. Example: if the current tag is `1.0.1`, the branch is `hotfix_1.0.1` and the release will be `1.0.2`.

## Step H1 — Create hotfix branch from develop

The branch is created from updated `develop`, but named after the current tag:

```bash
git checkout develop
git pull origin develop
git checkout -b hotfix_<CURRENT_TAG>
```

## Step H2 — Fix the issue

The user edits and commits on the `hotfix_<X.Y.Z>` branch. Assist with commands, don't make code changes for them:

```bash
git add <files>
git commit -m "fix: <short incident description>"
```

Confirm the fix was tested locally before proceeding.

## Step H3 — Merge hotfix back into develop

```bash
git checkout develop
git merge --no-ff hotfix_<CURRENT_TAG>
```

Don't push `develop` yet.

## Step H4 — Create release from develop and follow normal flow

The release uses the **next patch** (current tag + 1). From here, the flow is identical to steps 5–7 of the main workflow:

```bash
git checkout -b release_<NEXT_PATCH>

git checkout master
git pull origin master
git merge --no-ff release_<NEXT_PATCH>

git tag -a <NEXT_PATCH> -m "Merge branch 'release_<NEXT_PATCH>'"
git push origin develop master <NEXT_PATCH>

git checkout develop
```

The tag message follows the same format as a normal release: `"Merge branch 'release_<X.Y.Z>'"`.

## Full Example — current tag 1.0.1, login validation bug

```bash
# H0
git status --porcelain
git tag -l --sort=-v:refname | head -1     # → 1.0.1 (version with the issue)

# H1 — hotfix branch named after current tag
git checkout develop
git pull origin develop
git checkout -b hotfix_1.0.1

# H2 (user edits and commits)
git add src/auth/login.js
git commit -m "fix: login validation failed with uppercase emails"

# H3
git checkout develop
git merge --no-ff hotfix_1.0.1

# H4 — release with next patch (1.0.2)
git checkout -b release_1.0.2

git checkout master
git pull origin master
git merge --no-ff release_1.0.2

git tag -a 1.0.2 -m "Merge branch 'release_1.0.2'"
git push origin develop master 1.0.2

git checkout develop
```
