# Plan 003: Declarar cuándo NO usar la skill, para que deje de competir con code-review y planificadores

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 03369ee..HEAD -- skills/improve/SKILL.md README.md`
> Si alguno de esos archivos cambió desde que se escribió este plan, compara los
> extractos de "Estado actual" contra el código vivo antes de continuar; ante
> cualquier discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: S
- **Riesgo**: MED
- **Depende de**: ninguno
- **Categoría**: dx
- **Planificado en**: commit `03369ee`, 2026-08-17

## Por qué importa

El campo `description` del frontmatter es lo que un agente lee para decidir si
activa la skill. El de `improve` reclama nueve dominios: bugs, seguridad,
rendimiento, cobertura de tests, deuda técnica, migraciones, DX, docs y
sugerencias de features. No declara ni un solo caso negativo.

Esto no es un riesgo teórico. En la sesión donde se hizo este audit convivían
seis skills con solapamiento directo: `/code-review` (revisar un diff o PR),
`/security-review` (revisión de seguridad de cambios pendientes),
`/simplify` (limpiezas de calidad sobre lo cambiado), `claude-mem:make-plan`
(planes de implementación por fases), `claude-mem:pathfinder` (auditoría de
arquitectura previa a un refactor) y `claude-mem:oh-my-issues` (clustering de
backlog). Un modelo que elige entre esas seis a partir de sus descripciones no
tiene ningún criterio de desempate frente a `improve`.

Las consecuencias van en las dos direcciones y ambas son caras. Si `improve` se
activa donde correspondía `/code-review`, el usuario paga un audit de repo
completo para revisar un diff de veinte líneas. Si no se activa donde
correspondía, la skill simplemente no se usa. El eje que las separa es nítido y
hoy no está escrito en ninguna parte: **`improve` audita un repositorio en
reposo y produce especificaciones; las otras revisan un cambio en vuelo o
ejecutan trabajo.**

El riesgo es MED, no LOW, porque tocar la `description` cambia el
comportamiento de activación de la skill de forma difícil de medir sin evals
—que este repo todavía no tiene. Por eso el plan agrega la disciplina negativa
sin recortar ninguno de los disparadores positivos existentes.

## Estado actual

Archivos relevantes:

- `skills/improve/SKILL.md` — frontmatter con `description` (línea 3) y cuerpo
  de la skill; la sección `## Invocation variants` empieza en la línea 107 y
  `## Tone of the output` en la 120.
- `README.md` — sección `## Usage` (líneas 21-35) con la lista de invocaciones.

**Description actual** (`SKILL.md:3`), en una sola línea:

```yaml
description: Survey any codebase as a senior advisor and produce prioritized, self-contained implementation plans for OTHER models/agents to execute. Strictly read-only on source code — never implements, fixes, or refactors anything itself. Use when asked to audit a codebase, find improvement opportunities (bugs, security, performance, test coverage, tech debt, migrations, DX), suggest features or where to take the project next (roadmap, product direction), or generate handoff plans for another agent to implement.
```

**Frontmatter completo** (`SKILL.md:1-8`), para que veas su forma exacta:

```yaml
---
name: improve
description: <la línea de arriba>
license: MIT
metadata:
  author: shadcn
  version: "1.0.0"
---
```

**Variante `branch`** (`SKILL.md:112`) — importante porque es el único punto
donde `improve` sí opera sobre un cambio en vuelo, y por lo tanto el matiz que
la nueva sección debe preservar:

```markdown
- `branch` → audit only the current working branch's changes: scope = files changed since the merge-base with the default branch (...)
```

**Convenciones del repo que debes respetar:**

- Repo **solo markdown**: sin build, sin tests, sin gestor de paquetes.
- El contenido de la skill está **en inglés**. Escribe en inglés todo lo que
  agregues a `SKILL.md` y `README.md`, aunque este plan esté en español.
- El frontmatter es YAML y `description` es **una sola línea física**. Partirla
  en varias líneas sin la sintaxis YAML correcta rompe el parseo de la skill:
  es el error crítico a evitar en este plan.
- Las secciones de `SKILL.md` usan `##` para nivel superior y prosa densa sin
  relleno. Igualá esa densidad: sin viñetas de una palabra ni preámbulos.
- Estilo de commits: `docs: ...`, `fix: ...` en inglés imperativo.

## Comandos que vas a necesitar

Este repo no tiene build, tests, lint ni typecheck.

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 03369ee..HEAD -- <rutas en alcance>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -n '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Validez del frontmatter | ver Paso 1 | ejecutado en recon | `OK` |
| Estado del árbol | `git status --porcelain` | ejecutado en recon | solo archivos en alcance |

## Alcance

**En alcance** (los únicos archivos que debes modificar):
- `skills/improve/SKILL.md`
- `README.md`

**Fuera de alcance** (NO tocar, aunque parezcan relacionados):
- `.claude-plugin/plugin.json` — su `description` es texto de marketing para el
  catálogo, no un disparador de activación. Cambiarlo no afecta el ruteo y
  mezcla dos preocupaciones distintas en un mismo diff.
- `skills/improve/references/*.md` — los límites de uso son decisión de ruteo,
  no de procedimiento. Ningún reference file necesita cambiar.
- **Los disparadores positivos existentes de la `description`**. Este plan
  *agrega* una cláusula negativa; no recorta ninguno de los dominios ya
  listados. Recortarlos reduce la activación de forma que este repo todavía no
  puede medir.
- Cualquier renombre de la skill o del plugin — es una decisión de la fase C,
  con su propio plan.

## Flujo de git

- Rama: `advisor/003-usage-boundaries`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `docs: declare when not to use this skill`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 1: Agregar la cláusula negativa a la `description`

En `skills/improve/SKILL.md`, línea 3, **agrega al final** de la description
existente (sin borrar nada de lo que ya dice) esta oración:

```
Do NOT use for reviewing a diff, a PR, or uncommitted changes (that is a code-review skill's job), for implementing or refactoring anything, or for planning a feature the user has already scoped — this skill audits a repository at rest and produces specs for others to execute.
```

El resultado debe seguir siendo **una única línea física** de YAML. No la
partas, no agregues comillas, no introduzcas saltos de línea.

**Verificar** que el frontmatter sigue siendo YAML válido y que `description`
es un escalar de una línea:

```sh
awk 'NR==1&&/^---$/{f=1;next} f&&/^---$/{exit} f&&/^description:/{n++} END{print (n==1)?"OK":"FAIL"}' skills/improve/SKILL.md
```

→ `OK`. Y confirma que la línea 3 sigue siendo la description:
`sed -n '3p' skills/improve/SKILL.md | grep -c '^description:'` → `1`.

### Paso 2: Agregar la sección "When not to use this"

En `skills/improve/SKILL.md`, inserta una sección nueva **entre**
`## Invocation variants` y `## Tone of the output`. Contenido objetivo (inglés,
prosa densa al estilo del archivo):

```markdown
## When not to use this

The dividing line: this skill audits a repository **at rest** and produces
specifications. It does not review work in flight, and it does not do the work.

- **Reviewing a diff, a PR, or uncommitted changes** → a code-review skill. The
  one exception is the `branch` variant, which audits a branch's changes *as a
  body of work* and reports findings; it still writes plans rather than
  line-level review comments.
- **Implementing, fixing, or refactoring** → any implementation agent, or hand
  it a plan this skill already wrote. Hard Rule 5 already covers the direct ask;
  this is the routing version of it.
- **Planning something the user has already scoped** → a general planning skill.
  Use `plan <description>` here only when the specification benefits from a
  codebase investigation first.
- **Triaging or clustering an existing issue backlog** → an issue-triage skill.
  This one generates findings from source, it does not consume a tracker.

When two skills plausibly apply, prefer the narrower one. A full-repo audit is
an expensive way to answer a question scoped to twenty lines.
```

**Verificar**: `grep -c '^## When not to use this' skills/improve/SKILL.md`
→ `1`, y que quedó en la posición correcta:

```sh
awk '/^## Invocation variants/{i=NR} /^## When not to use this/{w=NR} /^## Tone of the output/{t=NR} END{print (i<w && w<t)?"OK":"FAIL"}' skills/improve/SKILL.md
```

→ `OK`.

### Paso 3: Reflejar el límite en el README

En `README.md`, agrega una sección corta después del bloque `## Usage`
(termina en la línea 35) y antes de `## How to use`:

```markdown
## When not to use it

`improve` audits a repository at rest and writes specs. For reviewing a diff or
a PR, use a code-review skill. For implementing, use an implementation agent —
or hand it one of the plans this wrote. For planning something you have already
scoped, a general planning skill is cheaper. `improve branch` is the one
in-flight case, and even there the output is findings and plans, not line
comments.
```

**Verificar**: `grep -c '^## When not to use it' README.md` → `1`.

## Plan de pruebas

Este repo no tiene suite de tests — es el hallazgo 10 del audit y está fuera del
alcance de este plan. No introduzcas un framework de testing.

La verificación es textual, y una de ellas es crítica: **el frontmatter debe
seguir parseando**. El comando `awk` del Paso 1 es el test de regresión que
protege contra el único fallo grave que este plan puede causar. Ejecútalo
después de cada paso, no solo después del Paso 1.

Verificación manual complementaria, a reportar en NOTES (no bloquea los done
criteria): relee la `description` resultante y confirma que ningún disparador
positivo original desapareció. Compárala contra el extracto de "Estado actual".

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] El comando `awk` del Paso 1 imprime `OK`
- [ ] `sed -n '3p' skills/improve/SKILL.md | grep -c '^description:'` → `1`
- [ ] `grep -c 'Do NOT use for reviewing a diff' skills/improve/SKILL.md` → `1`
- [ ] Los disparadores originales siguen presentes: `grep -c 'find improvement opportunities' skills/improve/SKILL.md` → `1` y `grep -c 'roadmap, product direction' skills/improve/SKILL.md` → `1`
- [ ] `grep -c '^## When not to use this' skills/improve/SKILL.md` → `1`
- [ ] El comando `awk` del Paso 2 imprime `OK`
- [ ] `grep -c '^## When not to use it' README.md` → `1`
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas, sin ninguna otra: `README.md`, `plans/README.md`, `skills/improve/SKILL.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- La `description` de `SKILL.md:3` no coincide con el extracto de "Estado
  actual" — el archivo derivó desde `03369ee`.
- El comando `awk` del Paso 1 imprime `FAIL` en cualquier momento: rompiste el
  frontmatter. Revierte ese paso inmediatamente y reporta; una skill con
  frontmatter inválido no carga.
- La description resultante supera los ~1200 caracteres. Algunos hosts truncan
  descripciones largas, y una cláusula negativa truncada es peor que ausente:
  reporta la longitud y espera instrucciones en vez de recortar por tu cuenta
  los disparadores positivos.
- Descubres que `## Invocation variants` o `## Tone of the output` ya no existen
  con esos títulos exactos: la sección nueva no tendría dónde anclarse.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Interacción futura**: esta sección y la `description` son ahora dos copias
  del mismo límite. Si el alcance de la skill cambia (por ejemplo si `branch`
  crece hacia review de PRs), ambas deben moverse juntas — y el README es una
  tercera copia.
- **Qué mirar en el review**: la validez del YAML por encima de todo, y que
  ningún disparador positivo se haya perdido al reescribir la línea. Un diff de
  una sola línea en el frontmatter es fácil de aprobar por inercia; léelo
  carácter por carácter.
- **Cómo saber si funcionó**: no se puede, todavía. Medir el efecto real sobre
  la activación requiere la suite de evals del hallazgo 10 (fase C), con casos
  positivos y negativos de ruteo. Hasta entonces esto es una mejora razonada, no
  una verificada — dilo así si alguien pregunta.
- **Deferido a propósito**: no se tocó `plugin.json`. Su descripción es de
  catálogo y no participa del ruteo; unificar ambos textos es cosmético y puede
  esperar al plan de versionado.
