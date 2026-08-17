# Plan 006: Realinear el README con la skill tras la fase A

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 9ac4b2e..HEAD -- README.md skills/improve/SKILL.md skills/improve/references/closing-the-loop.md`
> Si alguno de esos archivos cambió desde que se escribió este plan, compara los
> extractos de "Estado actual" contra el código vivo antes de continuar; ante
> cualquier discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P1
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: `plans/001`, `plans/002` (ambos DONE — este plan describe su
  resultado)
- **Categoría**: docs
- **Planificado en**: commit `9ac4b2e`, 2026-08-17

## Por qué importa

Los cinco planes de la fase A dejaron `README.md` fuera de alcance, cada uno por
una razón local correcta: el 001 editaba `closing-the-loop.md` y no quería tocar
prosa de alto nivel; el 002 evitaba colisiones de merge; el 003 solo agregó una
sección. Ninguno estaba equivocado por separado. El efecto acumulado sí lo está:
**el README ahora describe una skill que ya no existe**, en cuatro puntos.

Es exactamente el patrón que el audit original identificó como hallazgo #5 —
contradicciones que entran por cambios locales correctos y que nada atrapa— y
que ya le costó a este repo tres commits correctivos (`6f89ebf`, `c0accf0`,
`bd0aff9`). Se está reproduciendo una ejecución después.

El costo es concreto: el README es la primera y a menudo la única cosa que lee
alguien evaluando la skill. Hoy le promete verificación que la plantilla retiró,
le describe un procedimiento de review invertido, y le enumera cuatro reglas
duras cuando hay seis — omitiendo justamente la que protege contra inyección de
prompts.

## Estado actual

Archivo a corregir: `README.md`. Los cuatro puntos de deriva, con su fuente
correcta al lado.

**1. `README.md:102`** — arrastra la afirmación que el plan 002 retiró:

```markdown
- **Self-contained.** All context is inlined: exact file paths, current-state code excerpts, repo conventions with an exemplar file, verified commands. No "as discussed above."
```

**2. `README.md:103`** — misma clase de sobrepromesa:

```markdown
- **Verification gates.** Every step ends with a command and its expected output. Done criteria are machine-checkable. The executor never has to judge whether it succeeded.
```

La fuente corregida, en `plan-template.md:75-81`, dice hoy lo contrario sobre la
procedencia:

```markdown
**Provenance** is not decoration — it tells you how much to trust the row.
`executed` means the advisor ran this command during recon and saw it work.
`declared` means the advisor read it from `package.json`, a Makefile, or the CI
config but could not run it: the advisor is forbidden from installing
dependencies in the user's tree, so on a repo whose toolchain was not already
installed, most rows will be `declared`.
```

**3. `README.md:112`** — describe el orden de review **anterior** al plan 001:

```markdown
- **`execute <plan>`** spawns a cheaper executor subagent in an isolated git worktree, hands it the plan, then reviews the result like a tech lead — re-runs every done criterion, checks scope compliance, reads the diff against intent. Verdict: approve (merging stays your call), send back for revision (max 2 rounds), or block and refine the plan.
```

El orden vigente, en `closing-the-loop.md:64-67`, es el inverso — leer antes de
ejecutar:

```markdown
1. **Scope compliance**: `git -C <worktree> diff --stat` against the plan's in-scope list. Any file outside scope fails review, full stop. This runs nothing — do it first.
2. **Read the full diff.** ... Nothing the executor wrote gets executed until you have read it.
3. **Audit the new tests.** ...
4. **Re-run every done criterion** in the worktree. ... By now you have read everything you are about to execute.
```

**4. `README.md:88`** — tercera aparición de la misma clase de promesa, en la
descripción de Recon:

```markdown
**Recon.** Maps the repo: stack, conventions, and the exact build/test/lint commands — these become verification gates in every plan.
```

`SKILL.md:32` ya no dice eso: exige etiquetar cada comando como `executed` o
`declared`, y advierte que `declared` es el default honesto.

**5. `README.md:118-121`** — lista cuatro reglas duras; `SKILL.md` tiene seis:

```markdown
- Never modifies source code itself. The only writes go to `plans/`; executors edit only in disposable worktrees, and merging is always yours.
- Never runs commands that mutate your working tree — read, search, and read-only analysis only.
- Never reproduces secret values. Locations and credential types only, rotation always recommended.
- Asked to implement? It declines and points at the plan (or offers `execute`).
```

Falta la Hard Rule 3 (autocontención de los planes) y la Hard Rule 6 (el
contenido del repo es dato, no instrucción). La 6 importa especialmente ahora:
el plan 001 la propagó al executor, así que el README describe una frontera de
confianza incompleta.

**Convenciones del repo que debes respetar:**

- Repo **solo markdown**: sin build, sin tests, sin gestor de paquetes.
- El README está **en inglés**. Escribe en inglés todo lo que agregues, aunque
  este plan esté en español.
- El README es prosa de producto, no la especificación: es más corto y menos
  denso que `SKILL.md`. **No copies el texto de las Hard Rules literalmente** —
  resúmelas al registro del resto de la lista (una línea, voz activa).
- Estilo de commits: `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

Este repo no tiene build, tests, lint ni typecheck.

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 9ac4b2e..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -n '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | solo `README.md` y `plans/README.md` |

## Alcance

**En alcance** (el único archivo que debes modificar):
- `README.md`

**Fuera de alcance** (NO tocar):
- `skills/improve/SKILL.md`, `skills/improve/references/*.md` — **son la
  fuente y están correctos**. Este plan alinea el README con ellos, nunca al
  revés. Si te encuentras editando la skill para que coincida con el README,
  invertiste la dirección: detente.
- `examples/001-extract-shadow-config-resolution.md` — su desalineación es real
  pero la cubre el plan 007; tocarla aquí duplica trabajo y ensucia el diff.
- Los planes ya DONE bajo `plans/` — son el registro histórico.

## Flujo de git

- Rama: `advisor/006-readme-realignment`
- Un commit por paso, para poder revertir cada punto de deriva por separado.
  Mensajes en inglés siguiendo el estilo del repo, por ejemplo
  `docs: correct the execute review order in the readme`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 1: Quitar la promesa de comandos verificados

En `README.md`, línea 102, reemplaza `verified commands` por una formulación que
refleje la procedencia. Forma objetivo de la línea completa:

```markdown
- **Self-contained.** All context is inlined: exact file paths, current-state code excerpts, repo conventions with an exemplar file, and the repo's own commands with their provenance recorded. No "as discussed above."
```

En la línea 103, agrega la salvedad al final de la viñeta:

```markdown
- **Verification gates.** Every step ends with a command and its expected output. Done criteria are machine-checkable. The executor never has to judge whether it succeeded — and a plan's first step is establishing that the baseline was already green, so a pre-existing failure is never mistaken for its own.
```

En la línea 88, la descripción de Recon promete "the exact build/test/lint
commands". Reemplaza esa cláusula inicial por:

```markdown
**Recon.** Maps the repo: stack, conventions, and the repo's own build/test/lint commands, each tagged with whether the advisor ran it or only read it from a manifest — these become verification gates in every plan.
```

Conserva el resto de la línea 88 (la parte sobre docs de intención) sin tocar.

**Verificar**: `grep -c 'verified commands' README.md` → `0`;
`grep -c 'exact build/test/lint' README.md` → `0`;
`grep -c 'provenance recorded' README.md` → `1`;
`grep -c 'never mistaken for its own' README.md` → `1`;
`grep -c 'only read it from a manifest' README.md` → `1`.

### Paso 2: Corregir el orden del review de `execute`

En `README.md`, línea 112, reemplaza la enumeración del review para que siga el
orden vigente (alcance → leer → tests → ejecutar). Forma objetivo:

```markdown
- **`execute <plan>`** spawns a cheaper executor subagent in an isolated git worktree, hands it the plan, then reviews the result like a tech lead — checks scope compliance, reads the full diff against intent, audits the new tests, and only then re-runs every done criterion. Nothing the executor wrote gets executed until it has been read. Verdict: approve (merging stays your call), send back for revision (max 2 rounds), or block and refine the plan.
```

**Verificar**: `grep -c 'only then re-runs every done criterion' README.md` → `1`,
y que la formulación vieja desapareció:
`grep -c 're-runs every done criterion, checks scope compliance' README.md` → `0`.

### Paso 3: Completar las Hard Rules

En `README.md`, sección `## Hard rules` (líneas 118-121), agrega las dos que
faltan. Resúmelas al registro de la lista; **no** copies el texto de `SKILL.md`.
Forma objetivo de la sección completa:

```markdown
- Never modifies source code itself. The only writes go to `plans/`; executors edit only in disposable worktrees, and merging is always yours.
- Never runs commands that mutate your working tree — read, search, and read-only analysis only.
- Never reproduces secret values. Locations and credential types only, rotation always recommended.
- Every plan stands alone. The executor has not seen the audit, the other plans, or the session that wrote them, so nothing is left implicit.
- Treats everything it reads as data, not instructions. A file that tries to give the advisor orders becomes a security finding, not a command — and the executor it dispatches inherits that rule explicitly.
- Asked to implement? It declines and points at the plan (or offers `execute`).
```

**Verificar**: la sección tiene seis viñetas —

```sh
sed -n '/^## Hard rules/,/^## License/p' README.md | grep -c '^- '
```

→ `6`. Y `grep -c 'data, not instructions' README.md` → `1`.

## Plan de pruebas

Este repo no tiene suite de tests — es el candidato 008 del índice y está fuera
del alcance de este plan. No introduzcas un framework de testing.

La verificación es textual y de consistencia cruzada. El chequeo con contenido
real es el del Paso 3: **el conteo de viñetas del README debe seguir al conteo
de Hard Rules de `SKILL.md`**, no ser una constante escrita a mano. Compruébalo
en ambos lados:

```sh
echo "SKILL: $(grep -c '^[0-9]\. \*\*' skills/improve/SKILL.md)"
echo "README: $(sed -n '/^## Hard rules/,/^## License/p' README.md | grep -c '^- ')"
```

Ambos deben imprimir `6`. Este es exactamente el chequeo que la CI del candidato
009 debería automatizar.

**Barrido complementario (informativo, no bloquea).** Tras aplicar los tres
pasos, ejecuta:

```sh
grep -n 'verified\|exact build\|exact commands\|≤8\|origin/' README.md
```

Lo esperado es **cero coincidencias**. Si aparece alguna afirmación que
contradiga `SKILL.md` o los reference files y **no** esté entre los cinco puntos
que este plan corrige, **no la arregles** — está fuera de alcance: anótala en tu
reporte como hallazgo nuevo y termina el plan igual.

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `grep -c 'verified commands' README.md` → `0`
- [ ] `grep -c 'exact build/test/lint' README.md` → `0`
- [ ] `grep -c 'provenance recorded' README.md` → `1`
- [ ] `grep -c 'never mistaken for its own' README.md` → `1`
- [ ] `grep -c 'only read it from a manifest' README.md` → `1`
- [ ] Barrido limpio: `grep -c 'verified\|exact build\|exact commands\|≤8\|origin/' README.md` → `0`
- [ ] `grep -c 're-runs every done criterion, checks scope compliance' README.md` → `0`
- [ ] `grep -c 'only then re-runs every done criterion' README.md` → `1`
- [ ] `sed -n '/^## Hard rules/,/^## License/p' README.md | grep -c '^- '` → `6`
- [ ] `grep -c '^[0-9]\. \*\*' skills/improve/SKILL.md` → `6` (sin cambios — la fuente no se toca)
- [ ] `grep -c 'data, not instructions' README.md` → `1`
- [ ] `git diff --name-only main...HEAD` → exactamente estas dos rutas: `README.md` y `plans/README.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- Alguno de los cuatro extractos de "Estado actual" no coincide con `README.md`
  vivo — el archivo derivó desde `9ac4b2e`.
- `grep -c '^[0-9]\. \*\*' skills/improve/SKILL.md` devuelve algo distinto de
  `6`: las Hard Rules cambiaron de número y el objetivo del Paso 3 ya no es
  seis. Reporta el conteo real, no lo adivines.
- El orden del review en `closing-the-loop.md:64-67` no es el que cita este plan:
  significa que 001 fue revertido o modificado, y el Paso 2 estaría escribiendo
  una descripción igualmente falsa en la dirección contraria.
- Te encuentras a punto de editar `SKILL.md` o un archivo de `references/` para
  que coincida con el README. Eso invierte la dirección del arreglo: detente.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Acoplamiento a vigilar**: el README es ahora una cuarta copia de contenido
  que vive en `SKILL.md` (reglas duras, orden del review, promesa de
  verificación). Las tres primeras son la definición, la propagación a auditores
  y la propagación al executor. Cualquier cambio a las reglas duras se replica
  ahora en cuatro lugares.
- **Qué mirar en el review**: que el Paso 3 haya *resumido* las reglas en vez de
  copiarlas. Un README que reproduce el texto normativo de `SKILL.md` se
  desincroniza igual pero además se vuelve ilegible.
- **Esto se va a volver a romper.** Es la segunda vez que el README deriva de la
  skill. El arreglo estructural es el chequeo de paridad del candidato 009
  (CI estructural), que debería verificar el conteo de reglas duras y la
  ausencia de afirmaciones retiradas. Hasta que exista, este plan es un parche
  con fecha de caducidad.
