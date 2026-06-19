---
name: git-flow
description: Flujo Gitflow del equipo para releases, hotfixes y rollbacks.
---

Flujo Gitflow del equipo. Todos los merges usan `--no-ff`. Ejecuta **paso a paso, pidiendo confirmación al usuario antes de cada comando que modifique el repo o los remotos**.

## Interacción con el usuario

Presenta decisiones como **opciones seleccionables** (Tab), nunca preguntas abiertas. Solo usa texto libre cuando la respuesta no pueda enumerarse (ej. número de ticket).

## Convenciones de ramas

| Rama | Propósito |
|------|-----------|
| `master` | Producción |
| `develop` | Integración |
| `feature_ts<NUM>` | Por ticket (multi-ticket: `feature_ts<NUM1>_ts<NUM2>`) |
| `develop_ts<NUM>` | QA por ticket — se despliega a sandbox tras push del feature |
| `release_<X.Y.Z>` | Release |
| `hotfix_<X.Y.Z>` | Hotfix (sale de `develop`, no de `master`) |

## Antes de empezar

Recopila: **ticket(s)**, **otros features en el release**, **versión** (si no la da, propón en Paso 4).

Valida repo limpio, remotos alcanzables, ramas `develop_ts<NUM>` existen:

```bash
git rev-parse --show-toplevel && git remote -v && git status --porcelain && git fetch --all --prune
git branch -a | grep develop_ts<NUM>
```

Si algo falla, **detente** y avisa al usuario.

## Flujo principal

**Paso 1** — Crear feature desde develop:
`git checkout develop && git pull origin develop && git checkout -b feature_ts<NUM>`

**Paso 2** — Integrar develop_ts al feature y pushear (deploy a sandbox):
`git merge --no-ff develop_ts<NUM> && git push origin feature_ts<NUM>`
Multi-ticket: mergea cada `develop_ts<NUM>` antes del push. **Pausa hasta que QA valide en sandbox.**

**Paso 3** — Mergear feature(s) a develop (sin push aún):
`git checkout develop && git merge --no-ff feature_ts<NUM>`

**Paso 4** — Crear release (propón patch por defecto, confirma con usuario):
`git tag -l --sort=-v:refname | grep -E '^[0-9]+\.[0-9]+\.[0-9]+$' | head -1`
`git checkout -b release_<X.Y.Z>`

**Paso 5** — Mergear release a master:
`git checkout master && git pull origin master && git merge --no-ff release_<X.Y.Z>`

**Paso 6** — Tag y push (las tres referencias juntas):
`git tag -a <X.Y.Z> -m "Merge branch 'release_<X.Y.Z>'" && git push origin develop master <X.Y.Z>`

**Paso 7** — Volver a develop: `git checkout develop`

## Conflictos de merge

No abortes sin avisar. Muestra `git status`, ofrece opciones: resolver manual / abortar / dejar en conflicto. Tras resolver: `git add` + `git commit` (sin `-m`, mensaje de merge por defecto). No avances al siguiente paso hasta que quede limpio.

## Flujos adicionales

Lee el archivo correspondiente solo cuando lo necesites:

- **Hotfix**: `references/hotfix.md`
- **Rollback**: `references/rollback.md`
- **Ejemplos end-to-end**: `references/examples.md`

## Checklist final

Confirma: develop_ts integradas → feature pusheado y QA validó → develop con todos los features → release existe → master con merge del release → tag anotado (`-a`) → push de `develop master <tag>` juntos → usuario en `develop`.
