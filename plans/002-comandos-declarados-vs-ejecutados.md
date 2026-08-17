# Plan 002: La plantilla deja de afirmar que los comandos fueron verificados y registra su procedencia

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 03369ee..HEAD -- skills/improve/references/plan-template.md skills/improve/SKILL.md`
> Si alguno de esos archivos cambió desde que se escribió este plan, compara los
> extractos de "Estado actual" contra el código vivo antes de continuar; ante
> cualquier discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P1
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: ninguno
- **Categoría**: tech-debt
- **Planificado en**: commit `03369ee`, 2026-08-17

## Por qué importa

La propuesta de valor entera de esta skill son *verification gates*: cada paso
de cada plan termina en un comando con su resultado esperado, para que un modelo
ejecutor débil nunca tenga que **juzgar** si tuvo éxito. La plantilla lo dice
como propiedad número 2 de tres.

Esa promesa descansa sobre una afirmación falsa por construcción. La Hard Rule 2
prohíbe instalar dependencias. La plantilla, en cambio, cierra su tabla de
comandos afirmando que fueron *"verified during recon, not guessed"*. En un clon
recién hecho, sin `node_modules`, sin entorno virtual, sin toolchain, ninguno de
esos comandos se pudo ejecutar. El asesor los leyó de `package.json` o del
archivo de CI y los transcribió — que es lo correcto y lo único posible, pero no
es verificar.

La skill ya sabe que esto falla: `closing-the-loop.md:50` advierte que el
executor tendrá que instalar dependencias y quizá construir *"even though the
plan's command table (recon'd in the main tree) didn't mention it"*. Es decir,
parchea el síntoma en el momento del despacho y nunca corrige la plantilla que
lo origina.

El costo real: un executor que arranca el Paso 1 con el entorno sin preparar y
un comando que quizá no existe. Falla la primera verificación, no distingue "el
comando está mal" de "todavía no instalé nada", y —siendo un modelo débil—
improvisa. Cuando este plan aterrice, cada comando llevará su procedencia
declarada y todo plan arrancará con un paso de bootstrap cuya línea base roja es
condición de STOP, no un problema a resolver improvisando.

## Estado actual

Archivos relevantes:

- `skills/improve/references/plan-template.md` — la plantilla de plan; sección
  "Commands you will need" (líneas 65-74) y la lista de tres propiedades
  (líneas 5-9).
- `skills/improve/SKILL.md` — Hard Rule 2 (línea 19) y el bullet de Recon que
  exige los comandos exactos (línea 32).

**Sección actual de comandos** (`plan-template.md:65-74`), dentro del bloque de
plantilla:

```markdown
## Commands you will need

| Purpose   | Command                  | Expected on success |
|-----------|--------------------------|---------------------|
| Install   | `pnpm install`           | exit 0              |
| Typecheck | `pnpm typecheck`         | exit 0, no errors   |
| Tests     | `pnpm test -- <filter>`  | all pass            |
| Lint      | `pnpm lint`              | exit 0              |

(Exact commands from this repo — verified during recon, not guessed.)
```

Esa última línea entre paréntesis es la afirmación a corregir.

**Hard Rule 2** (`SKILL.md:19`), que hace imposible cumplirla:

```markdown
2. **Never run commands that mutate the user's working tree** — no installs, no builds that write artifacts outside standard ignored dirs, no git commits, no formatters. Read, search, and run read-only analysis only (e.g. `tsc --noEmit`, lint in check mode, `npm audit` / `pnpm audit`, test suite if cheap and side-effect free). Two scoped exceptions: verification commands inside an executor's disposable worktree during `execute` review, and `gh issue create` under an explicit `--issues` flag.
```

**Bullet de Recon** (`SKILL.md:32`):

```markdown
- Identify: language(s), framework(s), package manager, **how to build / test / lint / typecheck** (exact commands — these go into every plan as verification gates), test coverage shape, deployment target.
```

**Reconocimiento del problema río abajo** (`closing-the-loop.md:50`), que este
plan vuelve redundante pero **no** debes borrar (queda fuera de alcance):

```markdown
Note on fresh worktrees: they share git history but not `node_modules` or build artifacts — the executor must install dependencies first, and check tooling that resolves from `dist/` may need one build even though the plan's command table (recon'd in the main tree) didn't mention it. Expect this; it isn't a deviation.
```

**Convenciones del repo que debes respetar:**

- Repo **solo markdown**: sin build, sin tests, sin gestor de paquetes. La
  verificación es textual.
- `plan-template.md` contiene una plantilla **dentro de un bloque de código**
  (abre en la línea 17 con ```` ```markdown ```` y cierra en la 156). Los
  cambios de este plan van **dentro** de ese bloque, salvo donde se indique lo
  contrario. Confundir dentro/fuera del bloque es el error más fácil de cometer
  aquí.
- El contenido de la skill está **en inglés**. Escribe en inglés todo lo que
  agregues, aunque este plan esté en español.
- Estilo de commits: `fix: ...`, `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

Este repo no tiene build, tests, lint ni typecheck — no es una omisión del plan,
es la naturaleza del repo, y es precisamente el caso que este cambio enseña a
manejar con honestidad.

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 03369ee..HEAD -- <rutas en alcance>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -n '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Estado del árbol | `git status --porcelain` | ejecutado en recon | solo archivos en alcance |

## Alcance

**En alcance** (los únicos archivos que debes modificar):
- `skills/improve/references/plan-template.md`
- `skills/improve/SKILL.md` (una sola línea: el bullet 32)

**Fuera de alcance** (NO tocar, aunque parezcan relacionados):
- **La Hard Rule 2 en sí** (`SKILL.md:19`). Es correcta y es la razón de ser de
  este cambio: el asesor *no debe* instalar nada. Este plan adapta la plantilla a
  la regla, no la regla a la plantilla. Debilitar la regla para permitir
  installs sería exactamente el cambio equivocado.
- `skills/improve/references/closing-the-loop.md` — su nota sobre worktrees fríos
  queda redundante en parte, pero sigue siendo correcta y toca el flujo de
  `execute`, que el plan 001 está modificando. Evitar el conflicto.
- `examples/001-extract-shadow-config-resolution.md` — muestra congelada de un
  run real; no se re-escribe cuando cambia la plantilla.
- `README.md` — describe las verification gates en prosa; sigue siendo cierto.

## Flujo de git

- Rama: `advisor/002-command-provenance`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `fix: record command provenance instead of claiming verification`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 1: Agregar una columna de procedencia a la tabla de comandos

Dentro del bloque de plantilla de `plan-template.md`, reemplaza la sección
"Commands you will need" completa (líneas 65-74) por esta forma:

```markdown
## Commands you will need

| Purpose   | Command                  | Provenance | Expected on success |
|-----------|--------------------------|------------|---------------------|
| Install   | `pnpm install`           | declared   | exit 0              |
| Typecheck | `pnpm typecheck`         | declared   | exit 0, no errors   |
| Tests     | `pnpm test -- <filter>`  | declared   | all pass            |
| Lint      | `pnpm lint`              | executed   | exit 0              |

**Provenance** is not decoration — it tells you how much to trust the row.
`executed` means the advisor ran this command during recon and saw it work.
`declared` means the advisor read it from `package.json`, a Makefile, or the CI
config but could not run it: the advisor is forbidden from installing
dependencies in the user's tree, so on a repo whose toolchain was not already
installed, most rows will be `declared`. A `declared` command that fails is not
automatically your mistake — see Step 0.
```

Nota: el `executed` de la fila Lint es intencional; la tabla es un ejemplo y debe
mostrar ambos valores.

**Verificar**:
`grep -c 'verified during recon, not guessed' skills/improve/references/plan-template.md`
→ `0`, y
`grep -c '| Provenance |' skills/improve/references/plan-template.md` → `1`.

### Paso 2: Agregar un Paso 0 de bootstrap a la plantilla

En el bloque de plantilla, en la sección `## Steps`, inserta un paso previo
antes de `### Step 1: <imperative title>`:

```markdown
### Step 0: Establish a green baseline

Before changing anything, get the toolchain working and confirm the repo is
already healthy. Run the `Install` row, then every other command in the table
*as it exists on an unmodified checkout*.

- If they all pass: record that, and proceed to Step 1.
- If a `declared` command does not exist or fails on the unmodified checkout:
  that is a broken or mis-recorded baseline, not something you introduced.
  **STOP and report it** — include the command and its exact output. Do not
  "fix" the build to get moving, and do not proceed with a red baseline: you
  would have no way to tell your own regressions from pre-existing ones.
- If an `executed` command fails: the repo drifted since this plan was written.
  Treat it as a drift STOP condition.

**Verify**: every command in the table above runs and matches its expected
result on the unmodified checkout.
```

**Verificar**: `grep -c 'Step 0: Establish a green baseline'
skills/improve/references/plan-template.md` → `1`, y el string aparece **antes**
que `### Step 1:` en el archivo:

```sh
awk '/Step 0: Establish a green baseline/{s=NR} /^### Step 1:/{o=NR} END{print (s<o) ? "OK" : "FAIL"}' skills/improve/references/plan-template.md
```

→ `OK`.

### Paso 3: Agregar la condición de STOP correspondiente

En el bloque de plantilla, sección `## STOP conditions`, agrega un bullet a la
lista existente:

```markdown
- A command marked `declared` in the table does not exist or fails on an
  unmodified checkout (Step 0) — the baseline is broken; report it rather than
  repairing it.
```

**Verificar**: `grep -c 'does not exist or fails on an'
skills/improve/references/plan-template.md` → `1`.

### Paso 4: Actualizar la barra de calidad de la plantilla

Al final de `plan-template.md` está la sección `## Quality bar — check before
finishing each plan`, **fuera** del bloque de código. Agrega un bullet:

```markdown
- Does every row in the command table carry a `Provenance` value, and is
  `executed` claimed only for commands actually run during recon? A `declared`
  row is honest; a false `executed` is how an executor ends up debugging the
  advisor's guess.
```

**Verificar**: `grep -c 'is honest; a false'
skills/improve/references/plan-template.md` → `1`.

### Paso 5: Alinear el bullet de Recon en SKILL.md

En `skills/improve/SKILL.md`, línea 32, reemplaza el bullet por:

```markdown
- Identify: language(s), framework(s), package manager, **how to build / test / lint / typecheck** (exact commands — these go into every plan as verification gates, each tagged `executed` if you ran it during recon or `declared` if you only read it from a manifest or CI config; Hard Rule 2 forbids installing, so `declared` is the honest and expected default), test coverage shape, deployment target.
```

Es la **única** línea de `SKILL.md` que este plan modifica.

**Verificar**: `git diff --numstat main...HEAD -- skills/improve/SKILL.md` →
`1	1	skills/improve/SKILL.md` (una línea agregada, una borrada).

### Paso 6: Corregir el chequeo de alcance de la propia plantilla

La plantilla tiene el mismo defecto que este plan corrige en otro plano: un
comando de verificación que no verifica. En `plan-template.md`, dentro del
bloque de plantilla, sección `## Done criteria`, está esta línea:

```markdown
- [ ] No files outside the in-scope list are modified (`git status`)
```

Todo plan generado le pide al executor commitear su trabajo (sección `## Git
workflow`). Después de commitear, `git status` está limpio y ese criterio pasa
siempre, sin importar cuántos archivos fuera de alcance se hayan tocado.
Reemplázala por:

```markdown
- [ ] `git diff --name-only <base-branch>...HEAD` lists only the in-scope files
      (three dots — compares against the merge base, so it still holds if other
      work landed on the base branch meanwhile). `git status` is not a scope
      check here: this plan tells you to commit, and committed work leaves it
      clean.
```

**Verificar**: `grep -c 'is not a scope' skills/improve/references/plan-template.md`
→ `1`, y `grep -c 'are modified (`git status`)' skills/improve/references/plan-template.md`
→ `0`.

## Plan de pruebas

Este repo no tiene suite de tests — es el hallazgo 10 del audit y está fuera del
alcance de este plan. No introduzcas un framework de testing.

La verificación es textual y de consistencia:

- Los cinco greps de los pasos 1-5 dan el resultado esperado.
- La afirmación falsa desapareció del repo por completo (no solo de la plantilla).
- `SKILL.md` cambió exactamente una línea.
- El bloque de código de la plantilla sigue balanceado: el conteo de líneas que
  empiezan con triple backtick en `plan-template.md` debe ser **par**.

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `grep -rc 'verified during recon, not guessed' skills/` → `0` en todos los archivos
- [ ] `grep -c '| Provenance |' skills/improve/references/plan-template.md` → `1`
- [ ] `grep -c 'Step 0: Establish a green baseline' skills/improve/references/plan-template.md` → `1`
- [ ] El comando `awk` del Paso 2 imprime `OK`
- [ ] `grep -c 'does not exist or fails on an' skills/improve/references/plan-template.md` → `1`
- [ ] `grep -c 'is honest; a false' skills/improve/references/plan-template.md` → `1`
- [ ] `grep -c 'declared' skills/improve/SKILL.md` devuelve al menos `1`
- [ ] `git diff --numstat main...HEAD -- skills/improve/SKILL.md` → `1	1	skills/improve/SKILL.md`
- [ ] `grep -c 'is not a scope' skills/improve/references/plan-template.md` → `1`
- [ ] Bloque de plantilla balanceado: `grep -c '^```' skills/improve/references/plan-template.md` devuelve un número **par**
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas, sin ninguna otra: `plans/README.md`, `skills/improve/SKILL.md`, `skills/improve/references/plan-template.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso, y después de
> commitear `git status` está limpio y no verifica nada. Es literalmente el
> defecto que corriges en el Paso 6.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- La sección "Commands you will need" o el bullet 32 de `SKILL.md` no coinciden
  con los extractos de "Estado actual" — el repo derivó desde `03369ee`.
- El conteo de triple-backtick de `plan-template.md` queda **impar** tras
  cualquier paso: rompiste el bloque de plantilla. Revierte ese paso y reporta.
- Descubres que agregar el Paso 0 obliga a renumerar pasos ya referenciados
  desde otros archivos (`grep -rn 'Step 1' skills/` para comprobarlo antes de
  editar).
- Te encuentras a punto de modificar la Hard Rule 2 para que el asesor pueda
  instalar dependencias. Eso invierte la intención del plan: detente y reporta.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Interacción futura**: si alguna vez se agrega un modo sandbox donde el
  asesor *sí* pueda instalar en un entorno desechable, `executed` pasaría a ser
  el default y el Paso 0 se volvería barato en vez de defensivo. El vocabulario
  `declared`/`executed` está pensado para sobrevivir a ese cambio: solo cambia
  cuál es el valor habitual.
- **Qué mirar en el review**: que los cambios hayan caído **dentro** del bloque
  de plantilla y no fuera. Un Paso 0 escrito fuera del bloque no llega a ningún
  plan generado y el cambio queda decorativo. Es el modo de fallo más probable.
- **Deferido a propósito**: no se tocó `closing-the-loop.md:50`. Su nota sobre
  worktrees fríos queda parcialmente redundante una vez que existe el Paso 0,
  pero consolidarlas colisiona con el plan 001, que está editando ese archivo.
  Conviene hacerlo en un plan posterior, cuando 001 haya aterrizado.
