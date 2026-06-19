# Rollback

Usa este flujo cuando hay que revertir cambios. La estrategia depende de **en qué punto del flujo estás** cuando decides hacer rollback. **Nunca reescribas historia de `master` o `develop` si ya hay push al remoto** — usa `git revert` (crea commits que deshacen) en lugar de `git reset --hard` (reescribe historia).

## Paso R0 — Identificar el escenario

Pregunta al usuario explícitamente **dónde están los cambios que quiere revertir** antes de proponer nada:

- **Escenario A**: aún no hubo push final. Todo está local.
- **Escenario B**: ya se pusheó el feature (Paso 2), pero todavía no se mergeó a develop/master.
- **Escenario C**: ya se pusheó a `develop` y `master` con el tag (Paso 6 completo). El release ya está en producción.

## Escenario A — Rollback local (antes del push final)

Como nada se subió, se puede deshacer destruyendo ramas locales sin consecuencias. Confirma con el usuario antes de borrar nada.

```bash
git checkout master

# Borrar las ramas creadas durante el flujo
git branch -D release_<X.Y.Z>
git branch -D feature_ts<NUM>              # o feature_ts<NUM1>_ts<NUM2>
# hotfix_<X.Y.Z> si aplica

# Resetear develop y master al estado del remoto
git checkout develop
git reset --hard origin/develop
git checkout master
git reset --hard origin/master
```

Si se creó el tag local pero no se pusheó:

```bash
git tag -d <X.Y.Z>
```

## Escenario B — Feature pusheado pero sin merge a develop/master

El feature remoto está "contaminado" pero no afectó develop ni producción. Dos opciones:

**Opción 1 (preferida):** dejar el feature como está y simplemente no seguir el flujo. Si el ticket volverá, se puede seguir trabajando sobre el mismo feature.

**Opción 2:** borrar el feature del remoto (solo si el equipo lo acuerda, porque QA ya pudo haberlo validado):

```bash
git push origin --delete feature_ts<NUM>
git branch -D feature_ts<NUM>
```

## Escenario C — Release ya en producción (el más delicado)

Cuando `master` ya tiene el merge del release y el tag ya está en el remoto, el rollback se hace **con `git revert`**, no reescribiendo historia. Hay que revertir tanto en `master` como en `develop`.

### C.1 — Identificar el merge commit del release en master

```bash
git checkout master
git pull origin master
git log --merges --first-parent -n 5 --oneline
# Identifica el commit "Merge branch 'release_<X.Y.Z>'" y anota su SHA
```

### C.2 — Revertir el merge en master

Para revertir un merge commit se usa `-m 1` (mantiene el primer padre, que es master antes del merge):

```bash
git checkout master
git revert -m 1 <SHA-del-merge>
```

### C.3 — Revertir también en develop

El mismo release se había mergeado a `develop` (Paso 3 del flujo principal). Hay que revertirlo ahí también para que `develop` no arrastre los cambios revertidos al próximo release.

```bash
git checkout develop
git pull origin develop
git log --merges --first-parent -n 10 --oneline
git revert -m 1 <SHA-del-merge-en-develop>
```

### C.4 — Crear un nuevo tag patch y pushear

**No borres el tag original** del remoto — deja la trazabilidad. Crea un nuevo tag patch que refleje el estado revertido:

```bash
git tag -a <X.Y.Z+1> -m "Revert of release_<X.Y.Z>: <razón>"
git push origin develop master <X.Y.Z+1>

git checkout develop                       # quedar en develop al terminar
```

Ejemplo: si el release problemático era `1.0.2`, el tag de rollback será `1.0.3`.

## Ejemplo completo — revertir release 1.0.2

```bash
# C.1
git checkout master
git pull origin master
git log --merges --first-parent -n 5 --oneline
# Salida: a1b2c3d Merge branch 'release_1.0.2'

# C.2
git revert -m 1 a1b2c3d

# C.3
git checkout develop
git pull origin develop
git log --merges --first-parent -n 10 --oneline
git revert -m 1 <SHA-merge-en-develop>

# C.4
git checkout master
git tag -a 1.0.3 -m "Revert of release_1.0.2: pagos duplicados en checkout"
git push origin develop master 1.0.3

git checkout develop                       # quedar en develop al terminar
```

## Advertencias importantes

- **Nunca hagas `git reset --hard` en `master` o `develop` si ya hay push al remoto.** Reescribe historia y rompe el repo para todo el equipo.
- **No borres tags ya publicados.** Crea un nuevo tag patch para reflejar el rollback en el historial.
- **Después de un rollback en producción, considera hacer un hotfix** para arreglar la causa raíz en lugar de dejar el release revertido sin seguimiento.
- **Si hay conflictos al hacer revert**, resuelve como cualquier conflicto de merge (ver la sección correspondiente del flujo principal) y continúa con `git revert --continue`.
