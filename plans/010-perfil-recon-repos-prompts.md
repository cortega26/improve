# Plan 010: Perfil de recon para repos de prompts y configuración de agentes

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 8455800..HEAD -- skills/improve/SKILL.md skills/improve/references/audit-playbook.md`
> Si algo cambió desde que se escribió este plan, compara los extractos de
> "Estado actual" contra el código vivo antes de continuar; ante cualquier
> discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: M
- **Riesgo**: LOW
- **Depende de**: `plans/009` (recomendado, no bloqueante — el ejemplo
  trabajado de este plan referencia `scripts/check.py`, que 009 crea; si 009
  todavía no aterrizó, ver la nota en el Paso 1)
- **Categoría**: direction
- **Planificado en**: commit `8455800`, 2026-08-17

## Por qué importa

La Fase 1 (Recon) de esta skill exige identificar lenguaje, framework, gestor
de paquetes y comandos exactos de build/test/lint/typecheck. Este mismo repo
—el que la skill audita en su propio ejercicio de dogfooding— no tiene nada de
eso: es markdown puro, sin código ejecutable en el sentido tradicional. El
audit original que dio origen a todos estos planes tuvo que **improvisar**
fuera del playbook para poder auditarlo, y lo dejó anotado explícitamente como
hallazgo #3: *"un perfil de recon alternativo... cerraría el hueco"*.

Esto no es un caso de borde de un solo repo. Skills, configuraciones de
agentes (`CLAUDE.md`/`AGENTS.md`), librerías de prompts y manifiestos de
plugin son una clase de repositorio en crecimiento — y son precisamente el
tipo de proyecto donde alguien invocaría `/improve` para pedir ayuda,
justamente porque no tienen tooling tradicional que les dé una señal de
calidad. Hoy, cuando la skill se encuentra con uno, no tiene una ruta
declarada: tiene que razonar desde cero cada vez, con el riesgo de que un
modelo más débil simplemente reporte "no se identificó lenguaje, no hay tests,
no se puede auditar" y se detenga ahí.

## Estado actual

**Bullet de Recon que asume una pila de código** (`SKILL.md:31`):

```markdown
- Identify: language(s), framework(s), package manager, **how to build / test / lint / typecheck** (exact commands — these go into every plan as verification gates, each tagged `executed` if you ran it during recon or `declared` if you only read it from a manifest or CI config; Hard Rule 2 forbids installing, so `declared` is the honest and expected default), test coverage shape, deployment target.
```

**Frase de respaldo cuando no hay verificación** (`SKILL.md:37`), que asume el
caso "hay tests pero están rotos", no "no hay nada que pueda llamarse tests":

```markdown
If the repo has no working verification command (no tests, broken build), record that — "establish a verification baseline" is often finding #1, and it must precede risky plans in the dependency order.
```

**El playbook no tiene ninguna mención a repos sin pila de código** — las
nueve categorías (`audit-playbook.md`, líneas 9-104) están redactadas
asumiendo código fuente ejecutable en cada una: "Error handling", "Async
hazards", "N+1 patterns", "Missing indexes", etc.

**Ejemplo trabajado disponible tras el plan 009**: `scripts/check.py` +
`.github/workflows/check.yml` son ahora (si 009 ya aterrizó) el caso concreto
de "qué es infraestructura de verificación" para un repo de este tipo — un
checker de consistencia interna sin dependencias, no una suite de tests
tradicional.

**Convenciones del repo que debes respetar:**

- Repo **solo markdown** (más, tras el plan 009, un script Python stdlib y un
  workflow YAML). No introduzcas dependencias nuevas.
- El contenido de la skill está **en inglés**. Escribe en inglés todo lo que
  agregues, aunque este plan esté en español.
- Los archivos de `references/` siguen el patrón de los tres ya existentes
  (`audit-playbook.md`, `plan-template.md`, `closing-the-loop.md`): un
  `## Título` de nivel superior, secciones `##`/`###` debajo, tono directo sin
  relleno.
- Estilo de commits: `feat: ...`, `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 8455800..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -c '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| ¿Aterrizó 009? | `test -f scripts/check.py && echo SI || echo NO` | ejecutado en recon | ver Paso 1 |
| Si 009 aterrizó, el checker mismo | `python3 scripts/check.py` | declared (depende de 009) | `all checks passed` |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | ver criterios de done |

## Alcance

**En alcance**:
- `skills/improve/references/prompt-repo-recon.md` (archivo nuevo)
- `skills/improve/SKILL.md` (dos bullets en la Fase 1 Recon)
- `skills/improve/references/audit-playbook.md` (una nota corta, no una
  reescritura de las nueve categorías)

**Fuera de alcance** (NO tocar, aunque parezca relacionado):
- **Las nueve categorías del playbook, su contenido existente.** Este plan
  agrega una tabla de mapeo en el archivo nuevo, no reescribe
  `audit-playbook.md` categoría por categoría. Si te encuentras reescribiendo
  la sección `## 1. Correctness / Bugs` o cualquier otra, te saliste de
  alcance.
- `scripts/check.py`, `.github/workflows/check.yml` — son del plan 009; este
  plan solo los *referencia* como ejemplo, no los modifica.
- `plan-template.md`, `closing-the-loop.md` — no aplica ningún cambio de este
  plan a esos archivos.

## Flujo de git

- Rama: `advisor/010-prompt-repo-recon-profile`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `feat: add a recon profile for prompt and agent-config repos`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 0: Confirmar si el plan 009 ya aterrizó

```sh
test -f scripts/check.py && echo "009 aterrizado" || echo "009 no aterrizado"
```

Si **no** aterrizó, el Paso 1 más abajo tiene una nota entre corchetes sobre
qué texto usar en su lugar — no es una condición de STOP, es una variante
esperada según el orden en que se ejecuten los planes de la fase C.

### Paso 1: Crear el archivo de referencia nuevo

Crea `skills/improve/references/prompt-repo-recon.md` con este contenido
exacto:

```markdown
# Recon Profile — Prompt & Agent-Config Repositories

Read this during Phase 1 when recon turns up no language, no framework, no
package manager, and no build/test/lint stack to identify. That is not a
broken recon — it means the repo's content *is* the program. Skills, agent
instruction files (`CLAUDE.md`/`AGENTS.md`), prompt libraries, and plugin
manifests are executed by a model reading them, not by a compiler. This
skill's own repository is one: no `package.json`, no test runner, nothing to
`npm install` — `SKILL.md` and its `references/*.md` are the entire product.

Do not force the standard recon questions onto a repo shaped like this and
come back with "no language identified, no tests, unable to audit." Use this
profile instead.

## What "the program" is

- **Entry points**: `SKILL.md` files, `CLAUDE.md`/`AGENTS.md`, top-level
  prompt or system-message templates — whatever a host agent loads and
  executes as instructions.
- **Supporting files**: `references/*.md` (or equivalent) that entry points
  link to and expect to be read on demand; example/sample outputs kept as
  frozen artifacts; templates that generate other files.
- **Manifests**: `plugin.json`, `marketplace.json`, `package.json` for an
  npm-distributed skill, or any file a host platform reads to install or
  route to the skill. These usually duplicate a few facts also stated in the
  entry point (name, version, description) — that duplication is exactly
  where drift happens.

## What replaces build / test / lint

There is no compiler to catch a broken reference and no test runner to catch
a behavioral regression. What you can check mechanically:

- **Frontmatter validity** — YAML front matter parses, required fields
  (`name`, `description`) are present and non-empty.
- **Manifest validity and agreement** — every manifest is well-formed
  (`JSON.parse` / `json.load` succeeds), and fields that are supposed to
  match across files actually do (a `name` repeated in `plugin.json` and the
  skill's frontmatter; a `version` repeated in both).
- **Link resolution** — every relative markdown link (`[text](path)`,
  skipped inside fenced code blocks where they're usually illustrative, not
  navigational) resolves to a file that exists. A file rename silently
  breaks the skill for every installer if nothing catches it.
- **Surface parity** — every capability named in one user-facing surface
  (a README's usage table, a CLI help block) is also named in the entry
  point, and vice versa. Divergence here is the same failure class as code
  and its docs drifting apart, just with no compiler to force a fix.
- **Cross-file consistency of stated facts** — a count, a rule, or a limit
  stated in one file must match anywhere else it's restated. ("`≤8`
  concurrent subagents, one per category" against a category list of nine"
  is exactly this kind of bug — caught by reading, not by running anything.)

If none of this exists yet — no script, no CI step, nothing that runs these
checks mechanically — that absence **is finding #1**, same as "no working
verification command" is for a code repo. Frame it the same way: establishing
this baseline should be the first plan, ahead of anything else, because every
later plan needs a verification gate to point at.

**Worked example.** This skill's own repository had exactly this gap and
closed it: `scripts/check.py` (stdlib Python, no dependencies) plus
`.github/workflows/check.yml` now check frontmatter validity, manifest
agreement, link resolution, and variant parity on every push and PR. Point to
a structural checker like this as the concrete shape of "verification
infrastructure" when a repo has none — it is usually a few hundred lines and
does not require inventing a testing philosophy from scratch.

## How the nine audit categories map

Use this table instead of skipping categories that seem to assume a codebase.
Most still apply — the evidence just looks different.

| Category | What to look for in a prompt/config repo |
|---|---|
| Correctness / Bugs | Internal contradictions between files (the same rule stated two ways), off-by-one or miscounted facts, broken cross-references, stale line-number citations in one doc pointing at another. |
| Security | Secrets or tokens committed in example/config content, instructions embedded in any file that read as directives to an agent rather than data (prompt-injection shape — flag per this skill's own Hard Rule 6), credential-shaped strings in frozen example output. |
| Performance | Usually not applicable. Skip and say so, unless the repo does something like an expensive load-time chain (many chained file reads, a huge always-loaded context block that could be made on-demand). |
| Test Coverage | Reframe as: does *any* mechanical verification exist (a structural checker, a CI step)? If not, that's finding #1 — see above. If one exists, does it cover the failure modes that have actually recurred (check git history for repeated corrective commits)? |
| Tech Debt & Architecture | The same instruction or fact duplicated across three-plus files with no single source of truth; a doc that summarizes another doc and now disagrees with it; reference files nothing links to anymore. |
| Dependencies & Migrations | Usually thin. Still check: does any shipped script (a checker, a generator) have runtime dependencies, and are they pinned/documented? Is the repo's own distribution mechanism (npm package, plugin marketplace) declared consistently? |
| DX & Tooling | Missing CI entirely; no mechanical way for a contributor to know a PR is safe before a maintainer reads it by hand; no `CONTRIBUTING` explaining the repo's own conventions (which matter more here than in most codebases, since the "code" is prose). |
| Docs | Same as any repo — stale setup instructions, examples that no longer match current behavior. A frozen "sample output" file is not stale by definition (it's dated on purpose), but check whether readers can tell it's frozen and against what version. |
| Direction | Same as any repo — grounded suggestions from unfinished intent, stated-but-undelivered surface, asymmetries. |

## Plans for this repo shape

When writing plans against a prompt/config repo, the "Commands you will need"
table in the plan template still applies — but honestly reflects what exists.
Before any structural checker exists, most rows will legitimately be empty or
absent rather than `declared`/`executed`; do not invent a fake build/test/lint
row to fill the table. Once a checker exists (see the worked example above),
its invocation becomes the verification gate every subsequent plan in the repo
should use.
```

**Si el Paso 0 dio "009 no aterrizado"**: el párrafo "Worked example" de arriba
describe `scripts/check.py` en tiempo presente. Escríbelo igual — es una
afirmación sobre el estado *objetivo* del repo, no sobre este commit puntual, y
el plan 009 (independiente de este) lo hace cierto apenas aterrice. No agregues
condicionales tipo "once it lands" — el archivo debe leerse bien en cualquier
orden de aterrizaje.

**Verificar**: `test -f skills/improve/references/prompt-repo-recon.md &&
echo OK` → `OK`; `grep -c '^## How the nine audit categories map'
skills/improve/references/prompt-repo-recon.md` → `1`; y que la tabla de
mapeo tiene las nueve filas:
`grep -c '^| [A-Z]' skills/improve/references/prompt-repo-recon.md` → `10`
(9 categorías + la fila de encabezado `| Category | ... |`).

### Paso 2: Enganchar el archivo nuevo desde la Fase 1 de `SKILL.md`

En `SKILL.md`, línea 31 (el bullet de "Identify: language(s)..."), agrega una
oración al final que dirija a la ruta alternativa. Forma objetivo de la línea
completa:

```markdown
- Identify: language(s), framework(s), package manager, **how to build / test / lint / typecheck** (exact commands — these go into every plan as verification gates, each tagged `executed` if you ran it during recon or `declared` if you only read it from a manifest or CI config; Hard Rule 2 forbids installing, so `declared` is the honest and expected default), test coverage shape, deployment target. If none of language/framework/package manager/build stack apply — the repo is prompts, skills, or agent config rather than a codebase — stop here and read [references/prompt-repo-recon.md](references/prompt-repo-recon.md) instead of forcing these questions onto it.
```

En la línea 37 (la frase de respaldo sobre verificación), agrega una cláusula
que cubra el caso "nunca hubo nada que llamar tests", no solo "está roto".
Forma objetivo:

```markdown
If the repo has no working verification command (no tests, broken build — or, for a prompt/config repo, no structural checker at all), record that — "establish a verification baseline" is often finding #1, and it must precede risky plans in the dependency order.
```

**Verificar**: `grep -c 'read \[references/prompt-repo-recon.md\]' skills/improve/SKILL.md`
→ `1`; `grep -c 'or, for a prompt/config repo, no structural checker at all'
skills/improve/SKILL.md` → `1`.

### Paso 3: Nota corta en el playbook

En `audit-playbook.md`, inmediatamente después del párrafo de apertura (antes
de la línea horizontal `---` que precede a `## 1. Correctness / Bugs`), agrega
un párrafo corto:

```markdown
**Auditando un repo de prompts o configuración de agentes** (sin lenguaje,
framework, o pila de build/test/lint que identificar)? Las nueve categorías de
abajo siguen aplicando, pero la evidencia se ve distinta — usa la tabla de
mapeo en [prompt-repo-recon.md](prompt-repo-recon.md) antes de descartar una
categoría por "no aplica".
```

**No reescribas ninguna de las nueve secciones existentes.** Este párrafo es
la única adición a este archivo.

**Verificar**: `grep -c 'usa la tabla de mapeo en' skills/improve/references/audit-playbook.md`
→ `1`; y que las nueve secciones siguen intactas:
`grep -c '^## [0-9]\. ' skills/improve/references/audit-playbook.md` → `9`.

## Plan de pruebas

Este repo no tiene suite de tests tradicional — es precisamente el tipo de
repo que este plan describe. La verificación es textual y de consistencia
cruzada, más un chequeo funcional real: **el archivo nuevo debe describir con
precisión el estado del repo que lo aloja**. Si 009 ya aterrizó, confirma que
`scripts/check.py` y `.github/workflows/check.yml` existen tal como el
"Worked example" del Paso 1 los describe — el archivo nuevo estaría
describiendo algo falso si esos archivos no existen y el texto afirma que sí.

Si 009 aterrizó, ejecuta el checker que acabás de documentar como ejemplo, para
confirmar que sigue funcionando después de tu cambio (no debería tocarlo, pero
es la comprobación más barata de que no rompiste nada):

```sh
python3 scripts/check.py 2>&1 | tail -3
```

Debe seguir terminando en `all checks passed` — este plan no toca ningún
archivo que el checker valide de forma que pudiera romperlo (agrega un archivo
nuevo en `references/`, que el checker no inspecciona salvo por sus links
entrantes, y esos links siguen resolviendo).

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `test -f skills/improve/references/prompt-repo-recon.md && echo OK` → `OK`
- [ ] `grep -c '^## How the nine audit categories map' skills/improve/references/prompt-repo-recon.md` → `1`
- [ ] `grep -c '^| [A-Z]' skills/improve/references/prompt-repo-recon.md` → `10`
- [ ] `grep -c 'read \[references/prompt-repo-recon.md\]' skills/improve/SKILL.md` → `1`
- [ ] `grep -c 'or, for a prompt/config repo, no structural checker at all' skills/improve/SKILL.md` → `1`
- [ ] `grep -c 'usa la tabla de mapeo en' skills/improve/references/audit-playbook.md` → `1`
- [ ] `grep -c '^## [0-9]\. ' skills/improve/references/audit-playbook.md` → `9` (sin cambios — no se tocó ninguna categoría)
- [ ] Si `scripts/check.py` existe: `python3 scripts/check.py` sigue terminando en `all checks passed`
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas: `skills/improve/SKILL.md`, `skills/improve/references/audit-playbook.md`, `skills/improve/references/prompt-repo-recon.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- Los extractos de "Estado actual" no coinciden con el código vivo — el repo
  derivó desde `8455800`.
- `audit-playbook.md` ya no tiene exactamente nueve categorías numeradas: el
  número en la tabla de mapeo del archivo nuevo (Paso 1) y el chequeo del Paso
  3 dependen de que sean nueve. Reporta el conteo real.
- Te encuentras reescribiendo cualquiera de las nueve secciones existentes del
  playbook, o el contenido de `scripts/check.py`/`check.yml`. Ambos están
  fuera de alcance.
- Si `scripts/check.py` existe y tu cambio hace que deje de pasar
  (`all checks passed` deja de aparecer): tu adición rompió un link o algo que
  el checker valida. No tiene sentido que pase — investigá antes de continuar,
  no fuerces el commit igual.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Interacción con 009**: si este plan se ejecuta antes de 009, el "Worked
  example" del archivo nuevo describe `scripts/check.py` en presente aunque
  todavía no exista en el commit donde aterriza este plan. Es intencional (ver
  Paso 1) — dentro de poco será cierto, y el archivo no debería necesitar un
  segundo commit solo para cambiar el tiempo verbal.
- **Qué mirar en el review**: que la tabla de mapeo de categorías tenga
  exactamente nueve filas de contenido (más el encabezado) y que ninguna
  categoría del playbook original haya sido tocada — es la forma más fácil de
  detectar si el executor se salió de alcance.
- **Deferred:** aplicar este perfil retroactivamente al audit original de este
  mismo repo (el que generó los planes 001-008) para ver si habría cambiado
  algún hallazgo. No bloqueado por nada — es simplemente trabajo que no se
  hizo porque el perfil no existía todavía cuando se corrió ese audit.
