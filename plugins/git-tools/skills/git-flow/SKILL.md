---
name: git-flow
description: Flujo Gitflow del equipo para releases y hotfixes.
allowed-tools: Bash
---

# Gitflow — Release y Hotfix

## Overview

Prepara ramas para liberar tickets a desarrollo, sandbox y producción siguiendo el flujo Gitflow del equipo. Todos los merges usan `--no-ff`.

## Convenciones de ramas

| Rama | Propósito |
|------|-----------|
| `master` | Producción |
| `develop` | Integración |
| `feature_ts<NUM>` | Por ticket (multi-ticket: `feature_ts<NUM1>_ts<NUM2>`) |
| `develop_ts<NUM>` | QA por ticket — se despliega a sandbox tras push del feature |
| `release_<X.Y.Z>` | Release |
| `hotfix_<X.Y.Z>` | Hotfix — sale de `develop`, se nombra con el tag actual (versión con el problema) |

## Detección automática

Cuando el usuario diga "prepara esta rama" (o similar), detecta el tipo de rama actual:

```bash
git branch --show-current
```

| Rama actual | Tipo detectado |
|-------------|----------------|
| `develop_ts<NUM>` | develop |
| `feature_ts<NUM>` | feature |
| `hotfix_<X.Y.Z>` | hotfix |

Según el tipo detectado, pregunta al usuario con opciones seleccionables **hasta dónde preparar**:

| Opción | Destino | Pasos |
|--------|---------|-------|
| `[Sandbox]` | Feature → push (QA valida, se despliega a sandbox) | develop_ts: 1–3 / feature_ts: 2–3 |
| `[Producción]` | Todo hasta master + tag | develop_ts: 1–7 / feature_ts: 2–7 |

**Si es `hotfix`:**
No preguntar — siempre va hasta producción. Lee `references/hotfix.md` y ejecuta desde H3.

**Después de la selección, ejecuta todos los pasos sin preguntar más.** Solo detente si hay un conflicto de merge.

## Workflow

### 1. Crear feature desde develop

```bash
git checkout develop
git pull origin develop
git checkout -b feature_ts<NUM>
```

Multi-ticket: usa `feature_ts<NUM1>_ts<NUM2>` en el orden que indique el usuario.

### 2. Integrar develop_ts al feature

```bash
git merge --no-ff develop_ts<NUM>
```

Multi-ticket: mergea cada `develop_ts<NUM>` correspondiente.

### 3. Push del feature (deploy a sandbox)

```bash
git push origin feature_ts<NUM>
```

Si el alcance es **hasta sandbox**, el flujo termina aquí.

### 4. Mergear feature(s) a develop

```bash
git checkout develop
git merge --no-ff feature_ts<NUM>
```

No hagas push de `develop` todavía.

### 5. Crear release

Obtén el último tag y calcula el siguiente patch automáticamente:

```bash
git tag -l --sort=-v:refname | grep -E '^[0-9]+\.[0-9]+\.[0-9]+$' | head -1
git checkout -b release_<X.Y.Z>
```

Usa **patch** por defecto (`1.0.0` → `1.0.1`).

### 6. Mergear release a master

```bash
git checkout master
git pull origin master
git merge --no-ff release_<X.Y.Z>
```

### 7. Tag y push

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

## Checklist final

**Hasta sandbox:**
- [ ] `develop_ts<NUM>` integradas al feature
- [ ] Feature pusheado

**Hasta producción:**
- [ ] `develop` tiene todos los features del release
- [ ] `release_<X.Y.Z>` existe
- [ ] `master` tiene el merge del release
- [ ] Tag anotado (`-a`) con mensaje `"Merge branch 'release_<X.Y.Z>'"`
- [ ] Push de `develop`, `master` y tag juntos
- [ ] Usuario en `develop`
