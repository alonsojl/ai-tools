# Ejemplos completos del flujo principal

## Ticket único — ts176, release 1.0.1

```bash
# Validación inicial
git status --porcelain                     # vacío
git fetch --all --prune
git branch -a | grep develop_ts176         # existe

# Paso 1
git checkout develop
git pull origin develop
git checkout -b feature_ts176

# Paso 2 (deploy a sandbox tras el push)
git checkout feature_ts176
git merge --no-ff develop_ts176
git push origin feature_ts176
# --- pausa: QA valida en sandbox ---

# Paso 3
git checkout develop
git merge --no-ff feature_ts176

# Paso 4
git tag                                    # último: 1.0.0
git checkout -b release_1.0.1

# Paso 5
git checkout master
git pull origin master
git merge --no-ff release_1.0.1

# Paso 6
git tag -a 1.0.1 -m "Merge branch 'release_1.0.1'"
git push origin develop master 1.0.1

# Paso 7
git checkout develop
```

## Múltiples tickets — ts176 + ts177, release 1.0.2

Incluye además un feature extra (`feature_tsABC`) que ya existía y entra en el mismo release.

```bash
# Paso 1
git checkout develop
git pull origin develop
git checkout -b feature_ts176_ts177

# Paso 2
git checkout feature_ts176_ts177
git merge --no-ff develop_ts176
git merge --no-ff develop_ts177
git push origin feature_ts176_ts177
# --- pausa: QA valida en sandbox ---

# Paso 3
git checkout develop
git merge --no-ff feature_ts176_ts177
git merge --no-ff feature_tsABC

# Paso 4
git tag                                    # último: 1.0.1
git checkout -b release_1.0.2

# Paso 5
git checkout master
git pull origin master
git merge --no-ff release_1.0.2

# Paso 6
git tag -a 1.0.2 -m "Merge branch 'release_1.0.2'"
git push origin develop master 1.0.2

# Paso 7
git checkout develop
```
