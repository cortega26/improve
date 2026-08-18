# Plan 011: Suite de evals propia — validación estructural ahora, casos de comportamiento pendientes de early access

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 281903a..HEAD -- .github/workflows/check.yml skills/improve/SKILL.md`
> Si algo cambió desde que se escribió este plan, compara los extractos de
> "Estado actual" contra el código vivo antes de continuar; ante cualquier
> discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: M
- **Riesgo**: LOW
- **Depende de**: `plans/009` (DONE — este plan extiende su `check.yml`)
- **Categoría**: tests
- **Planificado en**: commit `281903a`, 2026-08-17

## Por qué importa

Este audit `deep` se perdió dos contradicciones reales en la propia skill
—la propagación incompleta de las Hard Rules 4/6 al executor, y el orden
invertido del review en `execute`— que solo aparecieron al leer títulos de
PRs de otras personas en el upstream. Nada en este repo comprueba
mecánicamente que la skill *se comporta* como dice: que su `description` no
dispara en pedidos que deberían ir a otra skill, que sí dispara en pedidos de
auditoría, que respeta sus propias Hard Rules. `scripts/check.py` (plan 009)
valida consistencia estructural — links, frontmatter, manifiestos — pero no
prueba comportamiento.

**Investigación de esta sesión, verificada de forma directa, no asumida**:
Claude Code tiene un comando real para esto, `claude plugin eval`, pero está
**bloqueado por early access** en este entorno — confirmado ejecutándolo
(`claude plugin eval` → `` `plugin eval` is currently in early access ``,
exit 1) y también `claude plugin eval init --bare` (mismo mensaje, mismo exit
code). Esto significa que **no hay forma de ejecutar ni de generar la
plantilla canónica** de un caso de eval en este entorno — cualquier archivo
de eval que este plan escriba queda sin verificar contra una ejecución real
hasta que el gate se levante.

Hay, en cambio, un segundo comando relacionado que **sí es real y funciona
ahora mismo, sin gate**: `claude plugin validate <path> --strict` — un linter
estructural de manifiestos, confirmado localmente (`claude plugin validate .
--strict` → `✔ Validation passed`, exit 0). Es un validador distinto del
`scripts/check.py` de 009 (ese lee `plugin.json` con `json.load`;
`claude plugin validate` entiende el esquema completo de plugin/marketplace,
incluyendo campos que el checker casero no conoce) y no está wireado a CI
todavía.

Este plan entrega **dos cosas de confianza distinta, separadas con
honestidad**: la validación estructural nueva, verificada y en CI ahora
mismo; y el esqueleto de la suite de evals de comportamiento, escrito contra
la documentación disponible (`--help` de los dos comandos, más el texto de
ayuda del propio CLI) pero **explícitamente marcado como no verificado**
hasta que alguien con acceso al early access confirme que el esquema es
correcto.

## Estado actual

**Verificado en esta sesión, comandos reales, ejecutados directamente**:

```
$ claude --version
2.1.221 (Claude Code)

$ claude plugin validate . --strict
Validating marketplace manifest: /home/carlos/VS_Code_Projects/tools/improve-skill/.claude-plugin/marketplace.json

✔ Validation passed

$ claude plugin eval
`plugin eval` is currently in early access

$ claude plugin eval init --bare test-case
`plugin eval` is currently in early access

$ npm view @anthropic-ai/claude-code version
2.1.234
```

**`claude plugin eval --help`** (funciona pese al gate — el `--help` muestra
la documentación aunque el comando no ejecute), sintaxis confirmada:

```
Usage: claude plugin eval [options] [command] [target]

Run eval cases (evals/**/case.yaml or evals/**/prompt.md + graders/*.md) against
a plugin and report scored results.

Options:
  --allow-tools <tools...>  Operator grant for gated tools (Bash, Write, Edit, WebFetch, mcp__*)
  --case <glob>             Filter cases by name glob
  --json [path]             Print/write the full run result as JSON
  --judge-model <model>     Override LLM-grader model (default: haiku)
  --model <model>           Override model for all cases
  --report <path>           Write a self-contained HTML report
  --runs <n>                Override per-case runs (default: case.runs ?? 3)
  --scaffold                Run each case's scaffold_script (off by default)
  --tag <tag...>            Filter cases by tag (repeatable)
  --threshold <0..1>        Exit 1 if any case score is below this threshold (default: 1.0)

Commands:
  init [options] [name]     Author an eval suite under evals/ via an interview.
                            Use --bare <name> for a blank single-case template.
```

**`.github/workflows/check.yml` actual** (del plan 009, íntegro):

```yaml
name: check
on:
  push:
    branches: [main]
  pull_request:
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: python3 scripts/check.py
```

**`SKILL.md`'s "When not to use this"** (del plan 003) — la afirmación de
comportamiento que el caso de eval negativo de este plan intenta poner a
prueba una vez que el gate se levante:

```markdown
- **Reviewing a diff, a PR, or uncommitted changes** → a code-review skill. The
  one exception is the `branch` variant...
```

**Convenciones del repo que debes respetar:**

- Repo con CI ahora (plan 009): `.github/workflows/check.yml`,
  `scripts/check.py`, ambos stdlib/sin dependencias.
- El contenido de la skill y de CI está **en inglés**. Escribe en inglés todo
  lo que agregues, aunque este plan esté en español.
- Estilo de commits: `feat: ...`, `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 281903a..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Versión del CLI | `claude --version` | ejecutado en recon | `2.1.221 (Claude Code)` o superior |
| Validador estructural (real, sin gate) | `claude plugin validate . --strict` | ejecutado en recon | `✔ Validation passed`, exit 0 |
| Suite de evals (gated, confirmado) | `claude plugin eval` | ejecutado en recon | `` `plugin eval` is currently in early access ``, exit 1 — **esperado**, no es un fallo |
| Paquete npm del CLI (para CI) | `npm view @anthropic-ai/claude-code version` | ejecutado en recon | una versión (ej. `2.1.234`) |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | ver criterios de done |

## Alcance

**En alcance**:
- `.github/workflows/check.yml` (un paso nuevo agregado al job existente —
  **no** toques el paso `python3 scripts/check.py` que trajo el plan 009)
- `evals/README.md` (archivo nuevo)
- `evals/does-not-fire-on-code-review-request/prompt.md` (archivo nuevo)
- `evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md` (archivo nuevo)
- `evals/fires-on-repo-audit-request/prompt.md` (archivo nuevo)
- `evals/fires-on-repo-audit-request/graders/skill-invoked.md` (archivo nuevo)

**Fuera de alcance** (NO tocar, aunque parezca relacionado):
- **`scripts/check.py` y el paso de CI que lo invoca.** Son del plan 009,
  llegaron completos y probados; este plan solo *agrega* un paso nuevo al
  mismo job, no toca el existente.
- **Intentar "arreglar" el gate de early access.** No es un bug de este repo,
  es una restricción de la cuenta/entorno. No hay nada que hacer al respecto
  desde este plan.
- **Afirmar en cualquier lugar que los casos de eval "pasan" o "fueron
  verificados".** No lo fueron — ver Condiciones de STOP.
- `SKILL.md`, `audit-playbook.md`, cualquier archivo de `references/` — este
  plan no cambia el comportamiento de la skill, solo agrega infraestructura
  para probarlo más adelante.

## Flujo de git

- Rama: `advisor/011-eval-suite-skeleton`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `feat: validate plugin manifests in CI`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 0: Confirmar el estado del gate en tu entorno

```sh
claude --version
claude plugin validate . --strict
claude plugin eval
```

El primero y segundo deben tener éxito (versión impresa, `✔ Validation
passed` con exit 0). El tercero **debe fallar** con el mensaje de early
access — eso confirma que tu entorno tiene el mismo estado que el de recon,
no que algo está roto.

**Si el tercer comando en tu entorno *no* falla** (es decir, si tenés acceso
al early access): DETENTE y reportalo — no ejecutes evals reales dentro de
este plan igual. Es una condición de STOP explícita, ver más abajo, porque
este plan asume que no tenés acceso y su alcance está calibrado para eso.

**Verificar**: los tres comandos dan el resultado esperado.

### Paso 1: Agregar el validador estructural a CI

En `.github/workflows/check.yml`, agrega un segundo paso al job `check`
existente, **después** del paso que corre `scripts/check.py`, sin tocar ese
paso. Forma objetivo del archivo completo:

```yaml
name: check
on:
  push:
    branches: [main]
  pull_request:
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: python3 scripts/check.py
      - name: Install Claude Code CLI
        run: npm install -g @anthropic-ai/claude-code
      - name: Validate plugin and marketplace manifests
        run: claude plugin validate . --strict
```

**Verificar**: `grep -c 'claude plugin validate . --strict' .github/workflows/check.yml`
→ `1`; `grep -c 'python3 scripts/check.py' .github/workflows/check.yml` → `1`
(el paso de 009 sigue intacto); el YAML sigue siendo válido:

```sh
python3 -c "import sys; content = open('.github/workflows/check.yml').read(); import re; assert re.search(r'^name: check$', content, re.M), 'malformed'; print('YAML structure looks intact')"
```

(Esto no es un parser YAML real — este repo no tiene PyYAML disponible por
convención de cero-dependencias — es solo una comprobación de que no
truncaste el archivo. La validación real ocurre cuando GitHub Actions corre
el workflow en el próximo push, algo que este plan no puede simular
localmente sin instalar `act` u otra herramienta, fuera de alcance por Hard
Rule 2.)

### Paso 2: Escribir el esqueleto de la suite de evals

Crea `evals/README.md` con este contenido exacto:

```markdown
# Eval suite for the `improve` skill

Behavioral tests for the skill's routing (does it trigger on the right
requests and stay silent on the wrong ones) and, eventually, its adherence
to its own Hard Rules. Run with `claude plugin eval` — see below.

## Status: unverified, blocked on early-access enablement

As of `claude --version` 2.1.221, `claude plugin eval` (and its `init`
subcommand, which would generate the canonical case template) are gated
behind early access in this environment — both fail with `` `plugin eval` is
currently in early access `` and exit 1. The two cases in this directory were
written against the command's `--help` output and general documentation, not
against a successful run or a generated template. **Treat their exact YAML
schema as unverified** until someone with access runs `claude plugin eval
init --bare sanity-check` and diffs its output against the files here.

## Running the suite (once enabled)

```sh
claude plugin eval evals/does-not-fire-on-code-review-request
claude plugin eval evals/fires-on-repo-audit-request
```

Or all cases at once, from the plugin root:

```sh
claude plugin eval .
```

## What's covered

| Case | What it checks | Grounded in |
|---|---|---|
| `does-not-fire-on-code-review-request` | The skill does NOT trigger on a bare diff-review request | `SKILL.md`'s "When not to use this" section (plan 003) |
| `fires-on-repo-audit-request` | The skill DOES trigger on a full-repo audit request | `SKILL.md`'s frontmatter `description` |

## Known limitation

The eval sandbox loads only the plugin under test — so the negative case
proves the skill doesn't self-fire on an out-of-scope prompt in isolation,
not that it would lose a routing contest against a real competing
code-review skill loaded alongside it. That stronger test — multiple plugins
loaded together — is possible per the documented `plugins:` frontmatter
field but wasn't attempted here, since it couldn't be verified either.

## What's not covered yet

Hard Rule 1 (never edits source code) and Hard Rule 4 (never reproduces
secret values) would make strong eval cases — asserting the skill never
calls `Write`/`Edit` outside `plans/`, or never echoes a value matching a
credential pattern — but both need `--allow-tools Write Edit` and a scaffold
repo with a fixture secret, which raises the complexity past what could be
written without being able to test it. Left for a follow-up once the basic
two-case mechanism is confirmed working.
```

**Verificar**: `test -f evals/README.md && echo OK` → `OK`;
`grep -c 'Status: unverified, blocked on early-access enablement' evals/README.md`
→ `1`.

### Paso 3: Caso negativo — no debería disparar en un pedido de code review

Crea `evals/does-not-fire-on-code-review-request/prompt.md`:

```markdown
---
name: does-not-fire-on-code-review-request
tags: [routing, negative]
---
Review the diff on this branch and tell me if you see any bugs in the changes.
```

Crea `evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md`:

```markdown
---
type: tool_used
tool: Skill
input_match: '"skill"\s*:\s*"(?:[\w-]+:)?improve"'
min: 0
max: 0
---
The `improve` skill must not fire for a bare request to review a diff or
uncommitted changes — that's explicitly out of scope per SKILL.md's "When
not to use this" section, except through the `branch` invocation variant,
which was not requested here.
```

**Verificar**: `test -f evals/does-not-fire-on-code-review-request/prompt.md
&& test -f evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md
&& echo OK` → `OK`.

### Paso 4: Caso positivo — debería disparar en un pedido de auditoría

Crea `evals/fires-on-repo-audit-request/prompt.md`:

```markdown
---
name: fires-on-repo-audit-request
tags: [routing, positive]
---
Audit this repository for tech debt, security issues, and what to build next.
```

Crea `evals/fires-on-repo-audit-request/graders/skill-invoked.md`:

```markdown
---
type: tool_used
tool: Skill
input_match: '"skill"\s*:\s*"(?:[\w-]+:)?improve"'
min: 1
---
A full-repo audit request naming multiple of the skill's stated categories
(tech debt, security, direction) should trigger the `improve` skill.
```

**Verificar**: `test -f evals/fires-on-repo-audit-request/prompt.md &&
test -f evals/fires-on-repo-audit-request/graders/skill-invoked.md &&
echo OK` → `OK`.

## Plan de pruebas

**Esto es inusual y hay que decirlo con toda claridad en el reporte final**:
la única parte de este plan que tiene una prueba real es el Paso 1 (CI). Los
pasos 2-4 no tienen — y no pueden tener, en este entorno — una prueba de que
el contenido que escriben es correcto más allá de "el archivo existe y tiene
la forma que documentamos". Eso es una limitación real del entorno, no un
plan de pruebas flojo a propósito.

Verificación negativa que sí podés hacer (no bloqueante, pero vale la pena):
confirmá que el `input_match` regex de ambos graders (`'"skill"\s*:\s*"(?:[\w-]+:)?improve"'`)
es una expresión regular válida en Python, ya que es lo más cerca que podés
llegar de una sanidad sintáctica sin poder ejecutar el eval real:

```sh
python3 -c "import re; re.compile(r'\"skill\"\s*:\s*\"(?:[\w-]+:)?improve\"'); print('regex valida')"
```

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `claude plugin validate . --strict` → `✔ Validation passed`, exit 0 (sigue pasando después de tu cambio — no debería verse afectado, pero confirmalo)
- [ ] `grep -c 'claude plugin validate . --strict' .github/workflows/check.yml` → `1`
- [ ] `grep -c 'python3 scripts/check.py' .github/workflows/check.yml` → `1`
- [ ] `python3 scripts/check.py` → sigue en `all checks passed` (este plan no debería tocar nada que el checker valide, pero confirmalo)
- [ ] `test -f evals/README.md && echo OK` → `OK`
- [ ] `grep -c 'Status: unverified, blocked on early-access enablement' evals/README.md` → `1`
- [ ] `test -f evals/does-not-fire-on-code-review-request/prompt.md && echo OK` → `OK`
- [ ] `test -f evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md && echo OK` → `OK`
- [ ] `test -f evals/fires-on-repo-audit-request/prompt.md && echo OK` → `OK`
- [ ] `test -f evals/fires-on-repo-audit-request/graders/skill-invoked.md && echo OK` → `OK`
- [ ] `python3 -c "import re; re.compile(r'\"skill\"\s*:\s*\"(?:[\w-]+:)?improve\"')"` → sin error
- [ ] `git diff --name-only main...HEAD` → exactamente estas seis rutas: `.github/workflows/check.yml`, `evals/README.md`, `evals/does-not-fire-on-code-review-request/prompt.md`, `evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md`, `evals/fires-on-repo-audit-request/prompt.md`, `evals/fires-on-repo-audit-request/graders/skill-invoked.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- `claude plugin eval` **no** falla con el mensaje de early access en tu
  entorno (es decir, tenés acceso). No sigas con este plan tal como está
  escrito — reportalo, porque significa que podés (y deberías) verificar de
  verdad los dos casos de eval en vez de dejarlos marcados como no
  verificados, y el alcance de este plan cambiaría.
- `npm install -g @anthropic-ai/claude-code` falla en tu entorno de CI/local.
  No inventes un método de instalación alternativo — reportá el error exacto.
- Te encuentras escribiendo en tu reporte o en cualquier archivo que los
  casos de eval "pasan" o "fueron verificados". No lo hicieron — el gate lo
  impide. Decir lo contrario sería falso.
- `claude plugin validate . --strict` deja de pasar después de tu cambio al
  workflow — significa que tocaste algo que no debías, investigá antes de
  continuar.
- Los extractos de "Estado actual" no coinciden con el código vivo — el repo
  derivó desde `281903a`.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Esto es trabajo a medio verificar, y está bien que lo sea.** La mitad de
  este plan (Paso 1, CI) es sólida y verificada. La otra mitad (pasos 2-4,
  los casos de eval) es la mejor especificación posible sin acceso al
  feature — no la trates como "terminada", trátala como "lista para que
  alguien con early access la confirme".
- **Primer paso cuando alguien tenga acceso**: correr
  `claude plugin eval init --bare sanity-check` en un directorio descartable,
  comparar la forma exacta del `prompt.md` y `graders/*.md` que genera contra
  los archivos de este plan, y corregir cualquier diferencia de esquema antes
  de confiar en los puntajes que reporten.
- **Qué mirar en el review**: que el Paso 1 no haya tocado el paso existente
  de `scripts/check.py`, y que ningún archivo nuevo afirme falsamente haber
  sido verificado por una ejecución real.
- **Deferred:** los dos casos de eval propuestos para Hard Rule 1 (nunca
  escribe fuera de `plans/`) y Hard Rule 4 (nunca reproduce secretos), que
  `evals/README.md` ya deja anotados como "not covered yet". Bloqueados por
  lo mismo que todo este plan — el gate de early access — y además necesitan
  un scaffold con fixture repo que este plan no intentó diseñar por no poder
  probarlo.
- **Deferred:** la prueba de contienda de ruteo real (dos skills cargadas a
  la vez, usando el campo `plugins:` del frontmatter) que probaría que
  `improve` pierde correctamente contra una skill de code-review real, no
  solo que no se dispara en aislamiento. Documentado como limitación conocida
  en `evals/README.md`, no intentado por la misma razón que todo lo demás.
