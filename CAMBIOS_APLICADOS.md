# Registro de cambios aplicados al repositorio

**Repositorio:** ariadnagd/grupo1_actividad2_automatizacion
**Fecha:** 2026-07-02
**Referencia:** BRANCHING_STRATEGY.md

---

## Paso 1 — Creación de la rama `develop`

**Qué se hizo:**
Se creó la rama `develop` a partir del commit más reciente de `main` (SHA `19c1f7e`).

**Por qué:**
El modelo de branching definido en `BRANCHING_STRATEGY.md` requiere dos ramas permanentes: `main` (producción estable) y `develop` (integración continua). Todas las ramas de feature parten de `develop` y hacen merge de vuelta a `develop`.

**Cómo:**
API de GitHub (`POST /git/refs`) con el SHA del HEAD de `main`.

---

## Paso 2 — Subida de `BRANCHING_STRATEGY.md` a `develop`

**Qué se hizo:**
Se subió el archivo `BRANCHING_STRATEGY.md` (existente en el directorio local del proyecto) a la rama `develop` del repositorio remoto.

**Por qué:**
El documento define toda la estrategia de trabajo del equipo: nomenclatura de ramas, flujo de trabajo, política de PRs, reglas de commits y checklists. Debe vivir en el repo para que todos los miembros puedan consultarlo directamente.

**Cómo:**
API de GitHub (`PUT /contents/BRANCHING_STRATEGY.md`) con el contenido en Base64, apuntando a `develop`.

---

## Paso 3 — Creación de la plantilla de Pull Request

**Qué se hizo:**
Se creó el archivo `.github/pull_request_template.md` en la rama `develop`.

**Por qué:**
La sección 5.2 del `BRANCHING_STRATEGY.md` especifica que debe existir una plantilla con campos estandarizados: descripción, tipo de cambio, cambios principales, pasos para probar, checklist del autor e issues relacionados. GitHub la aplica automáticamente al abrir cualquier PR.

**Cómo:**
API de GitHub (`PUT /contents/.github/pull_request_template.md`) en `develop`.

---

## Paso 4 — Protección de la rama `main`

**Qué se hizo:**
Se aplicaron las siguientes reglas sobre `main`:

- Merge directo prohibido: solo via Pull Request.
- Mínimo **2 aprobaciones** requeridas.
- Aprobaciones invalidadas si se hace push de nuevos commits.
- Force pushes y eliminación de rama prohibidos.
- Reglas activas también para administradores.

**Por qué:**
Sección 9 del `BRANCHING_STRATEGY.md`. `main` representa producción estable y requiere el nivel de revisión más alto.

**Cómo:**
API de GitHub (`PUT /branches/main/protection`) con `required_approving_review_count: 2` y `enforce_admins: true`.

---

## Paso 5 — Protección de la rama `develop`

**Qué se hizo:**
Se aplicaron las siguientes reglas sobre `develop`:

- Merge directo prohibido: solo via Pull Request.
- Mínimo **1 aprobación** requerida.
- Aprobaciones invalidadas si se hace push de nuevos commits.
- Force pushes y eliminación de rama prohibidos.
- Reglas activas también para administradores.

**Por qué:**
`develop` es la rama de integración continua. Requiere revisión pero con menos fricción que `main`: una aprobación basta para mantener la velocidad del equipo.

**Cómo:**
API de GitHub (`PUT /branches/develop/protection`) con `required_approving_review_count: 1`.

---

## Estado final del repositorio

| Elemento | Estado |
|----------|--------|
| Rama `main` | Existía previamente; protegida con 2 aprobaciones |
| Rama `develop` | Creada desde `main` HEAD; protegida con 1 aprobación |
| `.github/pull_request_template.md` | Creado en `develop` |
| `BRANCHING_STRATEGY.md` | Subido a `develop` |
| `CAMBIOS_APLICADOS.md` | Este archivo, en rama `docs/cambios-aplicados` |

---

## Próximos pasos recomendados

1. Aprobar y mergear esta PR a `develop` (requiere 1 aprobación).
2. Cuando `develop` esté validado, abrir PR `develop -> main` para que los archivos queden en producción (requiere 2 aprobaciones).
3. Revocar y rotar el token de acceso personal usado en esta sesión.
4. Configurar CI/CD en GitHub Actions para añadir status checks automáticos en las protecciones de rama.
