---
name: git-flow
description: Flujo Gitflow del equipo para releases y hotfixes.
allowed-tools: Bash
---

# Gitflow — Release y Hotfix

## Overview

Prepara ramas para liberar tickets a desarrollo, sandbox y producción siguiendo el flujo Gitflow del equipo. Todos los merges usan `--no-ff`. Ejecuta paso a paso, pidiendo confirmación antes de cada comando que modifique el repo o los remotos.

## Convenciones de ramas

| Rama | Propósito |
|------|-----------|
| `master` | Producción |
| `develop` | Integración |
| `feature_ts<NUM>` | Por ticket (multi-ticket: `feature_ts<NUM1>_ts<NUM2>`) |
| `develop_ts<NUM>` | QA por ticket — se despliega a sandbox tras push del feature |
| `release_<X.Y.Z>` | Release |
| `hotfix_<X.Y.Z>` | Hotfix — sale de `develop`, se nombra con el tag actual (versión con el problema) |

## Workflow

### 1. Tipo de preparación

Pregunta al usuario con opciones seleccionables:

| Opción | Develop branches | Alcance |
|--------|-----------------|---------|
| `[A — Feature único a sandbox]` | Una `develop_ts` | Pasos 2–4 (feature + push) |
| `[B — Feature multi-ticket a sandbox]` | Varias `develop_ts` | Pasos 2–4 (feature + push) |
| `[C — Feature único a producción]` | Una `develop_ts` | Pasos 2–8 (hasta master + tag) |
| `[D — Feature multi-ticket a producción]` | Varias `develop_ts` | Pasos 2–8 (hasta master + tag) |

Después obtén del usuario:

- **Ticket(s)** a liberar (`ts176`, o `ts176 + ts177` para B/D)
- **Versión** del release (solo C/D — si no la da, propón en el paso 6)

### 2. Validar estado del repo

```bash
git rev-parse --show-toplevel
git remote -v
git status --porcelain
git fetch --all --prune
git branch -a | grep develop_ts<NUM>
```

Si algo falla, **detente** y avisa al usuario.

### 3. Crear feature desde develop

```bash
git checkout develop
git pull origin develop
git checkout -b feature_ts<NUM>
```

Multi-ticket: usa `feature_ts<NUM1>_ts<NUM2>` en el orden que indique el usuario.

### 4. Integrar develop_ts al feature y pushear

```bash
git merge --no-ff develop_ts<NUM>
git push origin feature_ts<NUM>
```

Multi-ticket (B/D): mergea cada `develop_ts<NUM>` antes del push. **Pausa hasta que QA valide en sandbox.**

Opciones A/B: el flujo termina aquí. Opciones C/D: continúa al paso 5.

### 5. Mergear feature(s) a develop (solo C/D)

```bash
git checkout develop
git merge --no-ff feature_ts<NUM>
```

No hagas push de `develop` todavía.

### 6. Crear release (solo C/D)

Obtén el último tag para calcular la siguiente versión:

```bash
git tag -l --sort=-v:refname | grep -E '^[0-9]+\.[0-9]+\.[0-9]+$' | head -1
```

Regla de incremento (confirma con el usuario antes de crear la rama):

| Último tag | Tipo | Siguiente versión |
|------------|------|-------------------|
| `1.0.0` | patch (por defecto) | `1.0.1` |
| `1.0.0` | minor (funcionalidad nueva) | `1.1.0` |
| `1.0.0` | major (breaking changes) | `2.0.0` |

```bash
git checkout -b release_<X.Y.Z>
```

### 7. Mergear release a master (solo C/D)

```bash
git checkout master
git pull origin master
git merge --no-ff release_<X.Y.Z>
```

### 8. Tag y push (solo C/D)

```bash
git tag -a <X.Y.Z> -m "Merge branch 'release_<X.Y.Z>'"
git push origin develop master <X.Y.Z>
git checkout develop
```

Las tres referencias (`develop`, `master`, tag) se suben en el mismo comando. Al terminar, dejar al usuario en `develop`.

## Conflictos de merge

No abortes sin avisar. Muestra `git status`, ofrece opciones: resolver manual / abortar / dejar en conflicto. Tras resolver: `git add` + `git commit` (sin `-m`, mensaje de merge por defecto). No avances al siguiente paso hasta que quede limpio.

## Flujos adicionales

Lee el archivo correspondiente solo cuando lo necesites:

- **Hotfix**: `references/hotfix.md`
- **Ejemplos end-to-end**: `references/examples.md`

## Interacción con el usuario

Presenta decisiones como **opciones seleccionables** (Tab), nunca preguntas abiertas. Solo usa texto libre cuando la respuesta no pueda enumerarse (ej. número de ticket).

## Checklist final

**Todas las opciones (A/B/C/D):**
- [ ] `develop_ts<NUM>` integradas al feature
- [ ] Feature pusheado y QA validó en sandbox

**Solo C/D (producción):**
- [ ] `develop` tiene todos los features del release
- [ ] `release_<X.Y.Z>` existe
- [ ] `master` tiene el merge del release
- [ ] Tag anotado (`-a`) con mensaje descriptivo
- [ ] Push de `develop`, `master` y tag juntos
- [ ] Usuario en `develop`
