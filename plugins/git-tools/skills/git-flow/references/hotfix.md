# Flujo de Hotfix

Usa este flujo cuando hay que resolver un incidente que ya está en producción y no se puede esperar al siguiente ciclo normal de release. **Variante del equipo: el hotfix sale de `develop`, no de `master`** (a diferencia del Gitflow clásico). Una vez resuelto, se integra de vuelta a `develop` y desde ahí se crea la rama `release_<X.Y.Z>` para seguir con el flujo de integración conocido.

## Paso H0 — Validaciones previas

Mismas validaciones que la sección "Validación inicial" del flujo principal (repo correcto, sin cambios sin commitear, remotos alcanzables). Además confirma con el usuario:

- **Versión del hotfix** — normalmente es el siguiente patch. Si el último tag es `1.0.1`, el hotfix será `1.0.2`.
- **Descripción corta del incidente** — se usa en el mensaje del tag al final.

## Paso H1 — Crear rama hotfix desde develop

```bash
git checkout develop
git pull origin develop
git checkout -b hotfix_<X.Y.Z>
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
git merge --no-ff hotfix_<X.Y.Z>
```

No hagas push de `develop` todavía.

## Paso H4 — Crear release desde develop y continuar el flujo normal

A partir de aquí el flujo es **idéntico a los Pasos 4, 5, 6 y 7 del flujo principal**:

```bash
git checkout develop
git tag
git checkout -b release_<X.Y.Z>

git checkout master
git pull origin master
git merge --no-ff release_<X.Y.Z>

git tag -a <X.Y.Z> -m "Hotfix <X.Y.Z>: <descripción corta>"
git push origin develop master <X.Y.Z>

git checkout develop                       # quedar en develop al terminar
```

El mensaje del tag para hotfix es diferente al de un release normal — menciona "Hotfix" y la descripción del incidente, para distinguirlo en el historial.

## Ejemplo completo — hotfix 1.0.2 (bug de validación de login)

```bash
# H0
git status --porcelain
git tag                                    # último: 1.0.1

# H1
git checkout develop
git pull origin develop
git checkout -b hotfix_1.0.2

# H2 (el usuario edita y commitea)
git add src/auth/login.js
git commit -m "fix: validación de login fallaba con emails en mayúsculas"

# H3
git checkout develop
git merge --no-ff hotfix_1.0.2

# H4
git checkout -b release_1.0.2

git checkout master
git pull origin master
git merge --no-ff release_1.0.2

git tag -a 1.0.2 -m "Hotfix 1.0.2: validación de login con emails en mayúsculas"
git push origin develop master 1.0.2

git checkout develop                       # quedar en develop al terminar
```
