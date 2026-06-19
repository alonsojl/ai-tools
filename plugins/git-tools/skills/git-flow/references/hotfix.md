# Flujo de Hotfix

Usa este flujo cuando hay que resolver un incidente que ya está en producción y no se puede esperar al siguiente ciclo normal de release. **Variante del equipo: el hotfix sale de `develop`, no de `master`** (a diferencia del Gitflow clásico). Una vez resuelto, se integra de vuelta a `develop` y desde ahí se crea la rama `release_<X.Y.Z>` para seguir con el flujo de integración conocido.

## Paso H0 — Validaciones previas

Mismas validaciones que la sección "Validación inicial" del flujo principal (repo correcto, sin cambios sin commitear, remotos alcanzables). Además confirma con el usuario:

- **Descripción corta del incidente** — se usa en el mensaje del tag al final.

Obtén el tag actual (la versión en producción donde surgió el problema):

```bash
git tag -l --sort=-v:refname | grep -E '^[0-9]+\.[0-9]+\.[0-9]+$' | head -1
```

La rama hotfix usa **el tag actual** (la versión con el problema). El siguiente patch se usa después para el release. Ejemplo: si el tag actual es `1.0.1`, la rama es `hotfix_1.0.1` y el release será `1.0.2`.

## Paso H1 — Crear rama hotfix desde develop

La rama sale de `develop` actualizado, pero se nombra con el tag actual:

```bash
git checkout develop
git pull origin develop
git checkout -b hotfix_<TAG_ACTUAL>
```

## Paso H2 — Resolver la incidencia

El usuario edita y commitea en la rama `hotfix_<X.Y.Z>`. Acompaña con los comandos, no apliques cambios de código por él:

```bash
git add <archivos>
git commit -m "fix: <descripción corta del incidente>"
```

Antes de seguir, confirma que la solución fue probada localmente.

## Paso H3 — Mergear hotfix de vuelta a develop

```bash
git checkout develop
git merge --no-ff hotfix_<TAG_ACTUAL>
```

No hagas push de `develop` todavía.

## Paso H4 — Crear release desde develop y continuar el flujo normal

El release usa el **siguiente patch** (tag actual + 1). A partir de aquí el flujo es idéntico a los pasos 6–8 del flujo principal:

```bash
git checkout -b release_<SIGUIENTE_PATCH>

git checkout master
git pull origin master
git merge --no-ff release_<SIGUIENTE_PATCH>

git tag -a <SIGUIENTE_PATCH> -m "Merge branch 'release_<SIGUIENTE_PATCH>'"
git push origin develop master <SIGUIENTE_PATCH>

git checkout develop
```

El mensaje del tag sigue el mismo formato que un release normal: `"Merge branch 'release_<X.Y.Z>'"`.


## Ejemplo completo — tag actual 1.0.1, bug de validación de login

```bash
# H0
git status --porcelain
git tag -l --sort=-v:refname | head -1     # → 1.0.1 (versión con el problema)

# H1 — rama hotfix con el tag actual
git checkout develop
git pull origin develop
git checkout -b hotfix_1.0.1

# H2 (el usuario edita y commitea)
git add src/auth/login.js
git commit -m "fix: validación de login fallaba con emails en mayúsculas"

# H3
git checkout develop
git merge --no-ff hotfix_1.0.1

# H4 — release con el siguiente patch (1.0.2)
git checkout -b release_1.0.2

git checkout master
git pull origin master
git merge --no-ff release_1.0.2

git tag -a 1.0.2 -m "Merge branch 'release_1.0.2'"
git push origin develop master 1.0.2

git checkout develop
```
