# Estrategia de Branching, Revisión y Merge de Pull Requests

> Proyecto: **Análisis Macroeconómico del Euro** · Grupo 1 · Actividad 2

---

## Índice

1. [Modelo de branching](#1-modelo-de-branching)
2. [Estructura de ramas](#2-estructura-de-ramas)
3. [Nomenclatura de ramas](#3-nomenclatura-de-ramas)
4. [Flujo de trabajo completo](#4-flujo-de-trabajo-completo)
5. [Política de Pull Requests](#5-política-de-pull-requests)
6. [Proceso de Code Review](#6-proceso-de-code-review)
7. [Estrategia de merge](#7-estrategia-de-merge)
8. [Resolución de conflictos](#8-resolución-de-conflictos)
9. [Protección de ramas](#9-protección-de-ramas)
10. [Reglas de commits](#10-reglas-de-commits)
11. [Checklists de referencia rápida](#11-checklists-de-referencia-rápida)

---

## 1. Modelo de branching

Se adopta **GitHub Flow** simplificado con una rama de integración (`develop`), apropiado para equipos pequeños con ciclos de entrega cortos y un proyecto en fase activa de desarrollo.

```
main ─────────────────────────────────────────────────► producción estable
  │
develop ──────────────────────────────────────────────► integración continua
  │
  ├── feature/...  (nuevas funcionalidades)
  ├── fix/...      (correcciones de bugs)
  ├── refactor/... (mejoras internas sin cambio de comportamiento)
  └── docs/...     (documentación)
```

**Por qué este modelo y no GitFlow completo:**
- El equipo es pequeño (grupo de trabajo académico/profesional reducido).
- No hay versiones paralelas en producción que mantener.
- Permite velocidad de iteración sin la sobrecarga de ramas `release/*` y `hotfix/*`.

---

## 2. Estructura de ramas

### Ramas permanentes

| Rama | Propósito | Merge directo permitido |
|------|-----------|------------------------|
| `main` | Código en producción, siempre desplegable | No — solo via PR desde `develop` |
| `develop` | Integración de features, base de todas las ramas de trabajo | No — solo via PR |

### Ramas temporales (vida útil máxima: 1 sprint / 2 semanas)

| Tipo | Cuándo usarla | Ejemplo |
|------|--------------|---------|
| `feature/` | Nueva funcionalidad o módulo | `feature/ingesta-bce-api` |
| `fix/` | Corrección de bug en `develop` o `main` | `fix/interpolacion-series-temporales` |
| `refactor/` | Limpieza interna, sin cambios de comportamiento externo | `refactor/separar-capa-analisis` |
| `docs/` | Solo cambios de documentación | `docs/arquitectura-capas` |

Las ramas temporales **se eliminan** tras el merge aprobado.

---

## 3. Nomenclatura de ramas

```
<tipo>/<id-ticket-o-descripcion-corta-en-kebab-case>
```

**Reglas:**
- Minúsculas y guiones (`-`), nunca espacios ni guiones bajos.
- Descripción en español o inglés, coherente con el idioma del equipo; **no mezclar**.
- Máximo 50 caracteres tras el prefijo.
- Si existe sistema de tickets (GitHub Issues, Jira, etc.), incluir el identificador: `feature/42-dashboard-correlaciones`.

**Ejemplos válidos:**
```
feature/pipeline-eurostat
feature/modelo-volatilidad-garch
fix/normalizacion-iso8601
refactor/routes-separar-responsabilidades
docs/readme-instrucciones-instalacion
```

**Ejemplos inválidos:**
```
feature/Mi Nueva Feature          ← espacios y mayúsculas
arreglo_bug                        ← sin prefijo de tipo
feature/cambios                    ← demasiado vago
```

---

## 4. Flujo de trabajo completo

### 4.1 Inicio de tarea

```bash
# Siempre partir de develop actualizado
git checkout develop
git pull origin develop

# Crear la rama de trabajo
git checkout -b feature/ingesta-bce-api
```

### 4.2 Durante el desarrollo

```bash
# Commits frecuentes y atómicos (ver sección 10)
git add <archivos-específicos>
git commit -m "feat(ingesta): conectar endpoint BCE para tasas de interés"

# Publicar la rama en remoto desde el primer día
git push -u origin feature/ingesta-bce-api
```

### 4.3 Mantener la rama actualizada con develop

Antes de abrir la PR (y periódicamente durante el desarrollo largo):

```bash
git fetch origin
git rebase origin/develop
# En caso de conflictos, resolverlos y continuar:
# git rebase --continue
```

> Se prefiere `rebase` sobre `merge` para mantener un historial lineal en las ramas de feature. **Nunca hacer rebase de `develop` ni `main`.**

### 4.4 Apertura de Pull Request

Una vez el trabajo está listo:
1. Hacer push final: `git push origin feature/ingesta-bce-api`
2. Abrir PR en GitHub desde la interfaz web o CLI.
3. Completar la plantilla de PR (ver sección 5.2).
4. Asignar revisor(es) y etiquetas correspondientes.

### 4.5 Tras la aprobación

El merge lo ejecuta el **autor de la PR** una vez aprobada. Ver sección 7 para el método.

### 4.6 Limpieza post-merge

```bash
# Eliminar rama remota (GitHub puede hacerlo automáticamente si está configurado)
git push origin --delete feature/ingesta-bce-api

# Eliminar rama local
git checkout develop
git branch -d feature/ingesta-bce-api
git pull origin develop
```

---

## 5. Política de Pull Requests

### 5.1 Cuándo abrir una PR

- Cuando la funcionalidad está **completa y probada localmente**.
- Si el trabajo está en progreso pero se quiere feedback temprano, abrir como **Draft PR** con el prefijo `[WIP]` en el título.
- Una PR = una unidad lógica de trabajo. Evitar PRs que mezclen features no relacionadas.

### 5.2 Plantilla de PR

Crear el archivo `.github/pull_request_template.md` en el repositorio con el siguiente contenido:

```markdown
## Descripción
<!-- Qué hace esta PR y por qué es necesaria -->

## Tipo de cambio
- [ ] Feature nueva
- [ ] Bug fix
- [ ] Refactor
- [ ] Documentación

## Cambios principales
- 
- 
- 

## Cómo probar
<!-- Pasos para verificar que los cambios funcionan -->
1. 
2. 

## Checklist autor
- [ ] El código sigue las convenciones del proyecto
- [ ] He ejecutado los tests existentes y pasan correctamente
- [ ] He añadido tests para el código nuevo (si aplica)
- [ ] He actualizado la documentación (si aplica)
- [ ] La rama está actualizada con `develop` (rebase reciente)

## Issues relacionados
Closes #
```

### 5.3 Tamaño y alcance de una PR

| Indicador | Recomendado | Señal de alerta |
|-----------|-------------|-----------------|
| Líneas modificadas | < 400 | > 800 líneas |
| Archivos modificados | < 10 | > 20 archivos |
| Días de vida de la rama | < 5 días | > 10 días |

Si la PR es inevitablemente grande, añadir una descripción detallada del razonamiento y dividir la revisión en commits lógicos.

### 5.4 Tiempo de respuesta esperado

| Acción | Plazo máximo |
|--------|-------------|
| Primera revisión tras asignación | 24 horas |
| Respuesta a comentarios del revisor | 24 horas |
| Aprobación final tras correcciones | 24 horas |

---

## 6. Proceso de Code Review

### 6.1 Quién revisa

- **Mínimo 1 aprobación** requerida para merge a `develop`.
- **Mínimo 2 aprobaciones** requeridas para merge a `main`.
- El autor de la PR **no puede aprobar su propio código**.
- Rotar revisores para distribuir conocimiento del sistema entre el equipo.

### 6.2 Qué revisar

Los revisores deben evaluar en este orden de prioridad:

**Corrección funcional**
- ¿El código hace lo que dice que hace?
- ¿Los casos borde están contemplados?
- ¿Los datos de series temporales se manejan correctamente (periodicidades, NaN, índices de tiempo)?

**Arquitectura y diseño**
- ¿El cambio respeta la separación de capas del sistema (ingesta / limpieza / análisis / presentación)?
- ¿Se reutiliza código existente en lugar de duplicar?
- ¿Las nuevas dependencias están justificadas?

**Calidad del código**
- ¿Los nombres de variables, funciones y módulos son descriptivos?
- ¿Las funciones tienen responsabilidad única (SRP)?
- ¿El código es legible sin comentarios redundantes?

**Seguridad**
- ¿Las credenciales de API están en `.env` y no hardcodeadas?
- ¿Los inputs externos (datos de APIs) se validan antes de procesarse?

**Tests**
- ¿Las funciones críticas de análisis tienen tests?
- ¿Los mocks de API son representativos de los datos reales?

### 6.3 Cómo dejar comentarios

| Tipo de comentario | Cuándo usarlo | Ejemplo de prefijo |
|-------------------|--------------|-------------------|
| Bloqueante (debe resolverse antes del merge) | Bug, problema de seguridad, violación de arquitectura | `[BLOQUEANTE]` |
| Sugerencia (no obligatoria) | Mejora opcional de legibilidad o estilo | `[SUGERENCIA]` |
| Pregunta / aclaración | El revisor no entiende la intención del código | `[PREGUNTA]` |
| Nitpick (cosmético, opcional) | Espaciado, nombre de variable menor | `[NIT]` |

**Reglas de comunicación en reviews:**
- Comentar el **código**, nunca a la persona. "Esta función hace X" en lugar de "No entiendes cómo funciona X".
- Si hay más de 3 comentarios bloqueantes, considera una sesión de pair review en vivo.
- El revisor aprueba cuando todos los bloqueantes están resueltos, aunque queden sugerencias abiertas.

### 6.4 Responsabilidad del autor tras la revisión

- Responder todos los comentarios, incluso los que no se implementan (explicar el porqué).
- Si se hacen cambios significativos, solicitar una segunda ronda de revisión.
- No hacer force push a la rama mientras hay una revisión activa.

---

## 7. Estrategia de merge

### 7.1 Feature → develop: Squash and Merge

```
develop:  A──B──C──────────────────► D (squash del feature)
                 \                  /
feature:          x──y──z──w──v──u
```

**Por qué Squash:** Mantiene el historial de `develop` limpio con un commit por feature. Los commits intermedios de trabajo (WIP, typos, ajustes) no contaminan la historia del proyecto.

**Cómo hacerlo en GitHub:** Botón "Squash and merge" en la interfaz de la PR.

**El mensaje del squash commit debe seguir Conventional Commits (ver sección 10).**

### 7.2 develop → main: Merge Commit (no squash)

```
main:    A──B──────────────────────► M (merge commit)
               \                    /
develop:        C──D──E──F──G──H──►
```

**Por qué Merge Commit:** Preserva el punto exacto de integración y permite revertir un release completo con un solo `git revert`. El historial de `develop` queda trazable en `main`.

**Este merge solo se hace cuando `develop` está estable y validado.**

### 7.3 Resumen de métodos por destino

| Origen | Destino | Método | Quién ejecuta |
|--------|---------|--------|---------------|
| `feature/*`, `fix/*`, `refactor/*`, `docs/*` | `develop` | Squash and merge | Autor de la PR |
| `develop` | `main` | Merge commit | Tech lead / responsable del equipo |

---

## 8. Resolución de conflictos

### 8.1 Principio fundamental

**El autor de la rama de feature es responsable de resolver los conflictos**, no el equipo de revisión ni el responsable de `develop`.

### 8.2 Procedimiento

```bash
# 1. Actualizar develop local
git fetch origin
git checkout develop
git pull origin develop

# 2. Volver a la rama de feature y hacer rebase
git checkout feature/mi-feature
git rebase origin/develop

# 3. Si hay conflictos, Git los marcará:
# Abrir los archivos conflictivos, resolver manualmente,
# luego marcar como resueltos:
git add <archivo-resuelto>
git rebase --continue

# 4. Push forzado (solo permitido en ramas de feature propias)
git push --force-with-lease origin feature/mi-feature
```

> `--force-with-lease` es más seguro que `--force`: falla si alguien más ha hecho push a la rama mientras tanto.

### 8.3 Conflictos frecuentes en este proyecto

| Archivo | Causa habitual | Estrategia de resolución |
|---------|---------------|--------------------------|
| `routes.py` | Dos devs añaden endpoints simultáneamente | Mantener ambos bloques; revisar orden de rutas |
| `models.py` | Cambios en estructura de datos compartidos | Consensuar el schema antes de mergear |
| `app.py` | Configuración de la app modificada en paralelo | Revisar imports y configuración conjuntamente |
| `requirements.txt` | Añadir dependencias distintas en paralelo | Incluir todas; verificar compatibilidad de versiones |

---

## 9. Protección de ramas

Configurar en **GitHub → Settings → Branches** las siguientes reglas:

### Rama `main`

- [x] Require a pull request before merging
- [x] Require approvals: **2**
- [x] Dismiss stale pull request approvals when new commits are pushed
- [x] Require status checks to pass before merging (si hay CI configurado)
- [x] Require branches to be up to date before merging
- [x] Do not allow bypassing the above settings (desactivar para admins también)
- [x] Restrict who can push to matching branches (solo tech lead)

### Rama `develop`

- [x] Require a pull request before merging
- [x] Require approvals: **1**
- [x] Dismiss stale pull request approvals when new commits are pushed
- [x] Require branches to be up to date before merging
- [x] Do not allow bypassing the above settings

---

## 10. Reglas de commits

Se adopta el estándar **[Conventional Commits](https://www.conventionalcommits.org/)**.

### Formato

```
<tipo>(<ámbito>): <descripción-corta-en-imperativo>

[cuerpo opcional — explica el POR QUÉ, no el qué]

[footer opcional — Issues, breaking changes]
```

### Tipos permitidos

| Tipo | Cuándo usarlo |
|------|--------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `refactor` | Cambio interno sin modificar comportamiento externo |
| `test` | Añadir o corregir tests |
| `docs` | Solo documentación |
| `chore` | Tareas de mantenimiento (deps, config, CI) |
| `perf` | Mejora de rendimiento |

### Ámbitos sugeridos para este proyecto

| Ámbito | Qué cubre |
|--------|-----------|
| `ingesta` | Capa de ingesta y conectores de APIs (BCE, Eurostat, Yahoo Finance) |
| `limpieza` | Capa de procesamiento y homologación de series temporales |
| `analisis` | Módulos de cálculo estadístico y modelado de volatilidad |
| `ui` | Capa de presentación, dashboards y gráficos |
| `api` | Endpoints de la API REST |
| `models` | Modelos de datos |
| `config` | Configuración, variables de entorno, `.env` |
| `deps` | Dependencias (`requirements.txt`) |

### Ejemplos

```bash
# Bien
git commit -m "feat(ingesta): implementar cliente asíncrono para API del BCE"
git commit -m "fix(limpieza): corregir interpolación lineal con índices temporales duplicados"
git commit -m "refactor(analisis): extraer cálculo de volatilidad GARCH a módulo independiente"
git commit -m "chore(deps): actualizar pandas a 2.2.0 por CVE-2024-XXXXX"

# Mal
git commit -m "arreglé cosas"
git commit -m "WIP"
git commit -m "cambios"
git commit -m "feat: muchas cosas nuevas y también arreglé el bug del modelo"
```

### Reglas adicionales

- Descripción corta: máximo **72 caracteres**, en imperativo presente ("añadir", "corregir", no "añadí", "corrijo").
- Un commit = un cambio lógico. No mezclar feat + fix en el mismo commit.
- Los commits de trabajo durante el desarrollo pueden ser libres; el Squash and Merge los consolida en un único commit limpio en `develop`.

---

## 11. Checklists de referencia rápida

### Checklist: Antes de abrir una PR

```
[ ] La rama parte de develop actualizado (git rebase origin/develop)
[ ] El código funciona localmente (probar el flujo completo de la capa modificada)
[ ] Tests existentes pasan sin errores
[ ] No hay credenciales ni secrets en el diff
[ ] requirements.txt actualizado si se añadieron dependencias
[ ] Plantilla de PR completada con descripción, pasos de prueba y checklist
[ ] Revisor(es) asignados
[ ] Labels aplicados (feature / bug / refactor / docs)
```

### Checklist: Antes de aprobar una PR (revisor)

```
[ ] El código hace lo que describe la PR
[ ] Se respeta la arquitectura en capas del proyecto
[ ] No hay lógica de negocio en routes.py (pertenece a la capa de análisis o modelos)
[ ] Las variables de entorno se leen desde config / .env, no están hardcodeadas
[ ] Los datos de APIs externas se validan antes de procesarse
[ ] El código es legible: nombres descriptivos, funciones con responsabilidad única
[ ] No hay duplicación de lógica existente en otro módulo
[ ] Todos los comentarios bloqueantes anteriores están resueltos
```

### Checklist: Antes de mergear develop → main

```
[ ] develop tiene al menos 2 aprobaciones del equipo
[ ] Todos los tests pasan en develop
[ ] El README refleja el estado actual del sistema
[ ] No hay PRs bloqueantes abiertas sobre develop
[ ] El equipo está informado del merge a main
```

---

## Diagrama de flujo resumido

```
                    ┌──────────────────────────────────┐
                    │         Inicio de tarea           │
                    └────────────┬─────────────────────┘
                                 │
                    git checkout develop && git pull
                                 │
                    git checkout -b <tipo>/<descripcion>
                                 │
                    ┌────────────▼─────────────────────┐
                    │    Desarrollo + commits atómicos  │
                    │    (Conventional Commits)         │
                    └────────────┬─────────────────────┘
                                 │
                    git rebase origin/develop
                                 │
                    ┌────────────▼─────────────────────┐
                    │    Abrir Pull Request a develop   │
                    │    (plantilla completa + revisor) │
                    └────────────┬─────────────────────┘
                                 │
              ┌──────────────────▼──────────────────────┐
              │            Code Review                   │
              │  [BLOQUEANTE] → autor corrige y re-pushea│
              │  [SUGERENCIA] → opcional                 │
              └──────────────────┬──────────────────────┘
                                 │ ≥ 1 aprobación
                    ┌────────────▼─────────────────────┐
                    │   Squash and Merge → develop      │
                    │   Eliminar rama de feature        │
                    └────────────┬─────────────────────┘
                                 │
                    (cuando develop está estable)
                                 │
                    ┌────────────▼─────────────────────┐
                    │   PR develop → main               │
                    │   ≥ 2 aprobaciones                │
                    │   Merge Commit                    │
                    └──────────────────────────────────┘
```

---

*Documento elaborado siguiendo las convenciones de Git estándar de la industria. Cualquier excepción a estas reglas debe consensuarse con el equipo y documentarse aquí.*
