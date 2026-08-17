# Plan 008: `reconcile` cosecha los diferidos de las notas de mantenimiento

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 9ac4b2e..HEAD -- skills/improve/references/closing-the-loop.md skills/improve/references/plan-template.md`
> Si alguno cambió desde que se escribió este plan, compara los extractos de
> "Estado actual" contra el código vivo antes de continuar; ante cualquier
> discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: ninguno
- **Categoría**: dx
- **Planificado en**: commit `9ac4b2e`, 2026-08-17

## Por qué importa

La plantilla le pide a cada plan que declare, en sus notas de mantenimiento,
"any follow-up explicitly deferred out of this plan (and why)". Es una buena
instrucción: obliga a decir en voz alta lo que se decidió no hacer, en vez de
dejarlo como deuda tácita.

**Pero nada vuelve nunca a leer esas notas.** El procedimiento de `reconcile`
—el único flujo cuyo trabajo es procesar lo que pasó desde la última sesión—
recorre los planes por estado (DONE, BLOCKED, IN PROGRESS, TODO) y no menciona
las notas de mantenimiento en ninguna rama. El resultado es que el sitio donde
la skill le pide al asesor que aparque trabajo a propósito es también el sitio
donde ese trabajo va a morir.

Este repo lo demuestra ahora mismo, con dos diferidos mutuamente bloqueados:

- `plans/001:274` difirió alinear la nota "Note on fresh worktrees…" de
  `closing-the-loop.md:50`, porque la corrección dependía de cómo aterrizara el
  Paso 0 del plan 002.
- `plans/002:349` difirió consolidar esa misma nota con el Paso 0, porque el
  plan 001 estaba editando ese archivo y habrían colisionado.

Ambos planes están DONE. Ambos diferidos están desbloqueados desde entonces. Un
`reconcile` los habría sacado a la superficie; el `reconcile` actual no los ve,
porque solo mira estados y criterios de done. Salieron a la luz en una revisión
manual, que es precisamente lo que el flujo debería hacer innecesario.

Cuando este plan aterrice, `reconcile` leerá los diferidos de los planes DONE,
comprobará si su bloqueo se levantó, y los reportará como trabajo ejecutable.

## Estado actual

Archivos relevantes:

- `skills/improve/references/closing-the-loop.md` — sección `## `reconcile` —
  keep `plans/` alive`, el procedimiento completo.
- `skills/improve/references/plan-template.md` — sección `## Maintenance notes`
  dentro del bloque de plantilla, donde se declaran los diferidos.

**Procedimiento actual de `reconcile`**, íntegro:

```markdown
## `reconcile` — keep `plans/` alive

Process what happened since the last session. Read `plans/README.md` and every plan file, then per status:

- **DONE** — spot-check that the done criteria still hold on the current HEAD (cheap ones only). Mark verified in the index. Don't delete plan files — they're the record.
- **BLOCKED** — read the reason. Investigate the underlying obstacle in the codebase. Either rewrite the plan around it (new number if the approach changed fundamentally, in-place refresh otherwise) or mark REJECTED with one line of rationale.
- **IN PROGRESS** (stale) — flag it to the user; an executor probably died mid-run. Check the worktree if one exists.
- **TODO** — run the drift check. If drifted: re-verify the finding still exists (it may have been fixed in passing), then refresh the "Current state" excerpts and `Planned at` SHA. If the finding is gone, mark REJECTED ("fixed independently").

Finish with a short report: what's verified done, what was refreshed, what's rejected, and what's executable right now.
```

Ninguna de las cuatro ramas menciona las notas de mantenimiento.

**Sección de notas de mantenimiento en la plantilla** (dentro del bloque de
plantilla), donde nace el diferido:

```markdown
## Maintenance notes

For the human/agent who owns this code after the change lands:

- What future changes will interact with this (e.g. "if pagination is added
  to this endpoint, the batching in step 2 must be revisited").
- What a reviewer should scrutinize in the PR.
- Any follow-up explicitly deferred out of this plan (and why).
```

La tercera viñeta pide el diferido en prosa libre, sin forma reconocible. Los
dos diferidos reales de este repo empiezan con `**Deferido a propósito**` — pero
eso fue una convención accidental de quien los escribió, no algo que la
plantilla pida, y en inglés sería otra cosa. **Sin un marcador estable no hay
nada que `reconcile` pueda buscar.**

**Convenciones del repo que debes respetar:**

- Repo **solo markdown**: sin build, sin tests, sin gestor de paquetes.
- El contenido de la skill está **en inglés**. Escribe en inglés todo lo que
  agregues, aunque este plan esté en español.
- `plan-template.md` contiene una plantilla **dentro de un bloque de código**
  (abre en la línea 17 con ```` ```markdown ```` y cierra en la 187). El cambio
  del Paso 1 va **dentro** de ese bloque. Confundir dentro/fuera es el error
  más fácil de cometer aquí.
- Estilo de commits: `feat: ...`, `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

Este repo no tiene build, tests, lint ni typecheck.

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 9ac4b2e..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -n '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Paridad de fences | `grep -c '^```' skills/improve/references/plan-template.md` | ejecutado en recon | número **par** |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | ver criterios de done |

## Alcance

**En alcance** (los únicos archivos que debes modificar):
- `skills/improve/references/plan-template.md`
- `skills/improve/references/closing-the-loop.md`

**Fuera de alcance** (NO tocar, aunque parezcan relacionados):
- **Los diferidos reales de `plans/001` y `plans/002`.** Este plan enseña a
  `reconcile` a encontrarlos; **no** ejecuta la consolidación que describen. Son
  planes DONE y su texto es el registro histórico. Reescribirlos para que usen
  el marcador nuevo falsificaría ese registro — el Paso 3 los usa como caso de
  prueba tal como están.
- `skills/improve/SKILL.md` — su línea sobre `reconcile` es un resumen de una
  frase que sigue siendo cierto; ampliarlo duplica la especificación.
- `README.md` — su descripción de `reconcile` también sigue siendo cierta.

## Flujo de git

- Rama: `advisor/008-reconcile-harvests-deferrals`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `feat: reconcile harvests deferred follow-ups from done plans`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 1: Darle forma reconocible al diferido en la plantilla

En `plan-template.md`, **dentro** del bloque de plantilla, reemplaza la tercera
viñeta de `## Maintenance notes` por una que exija un marcador estable:

```markdown
- Any follow-up explicitly deferred out of this plan. Write each one on its own
  line starting with `**Deferred:**` so `reconcile` can find it later, and name
  what unblocks it — another plan, a decision, or "nothing, it just wasn't worth
  doing now". A deferral nobody can test for is a deferral nobody will ever
  action.
```

**Verificar**: `grep -c 'starting with `\*\*Deferred:\*\*`'
skills/improve/references/plan-template.md` → `1`, y el bloque sigue balanceado:
`grep -c '^```' skills/improve/references/plan-template.md` devuelve un número
**par**.

### Paso 2: Agregar la cosecha al procedimiento de `reconcile`

En `closing-the-loop.md`, en la sección de `reconcile`, amplía la rama **DONE**
y agrega un paso posterior. Forma objetivo de la rama DONE:

```markdown
- **DONE** — spot-check that the done criteria still hold on the current HEAD (cheap ones only). Mark verified in the index. Don't delete plan files — they're the record. Then read the plan's "Maintenance notes" and collect every `**Deferred:**` line — see "Harvest the deferrals" below.
```

Y agrega esta subsección inmediatamente antes del párrafo final
("Finish with a short report…"):

```markdown
### Harvest the deferrals

The maintenance notes are where the advisor deliberately parked work, and
nothing else in this flow reads them. After walking the statuses, collect every
`**Deferred:**` line from the plans now marked DONE and decide each one:

- **Unblocked** — whatever it was waiting on has landed. Report it as
  executable work, with the plan and line it came from. If it is substantial,
  it earns its own plan on the next planning run; if it is a one-line cleanup,
  say so and let the user decide.
- **Still blocked** — name what it is waiting on, and carry it forward. A
  deferral blocked on a plan that was later REJECTED is not blocked any more —
  it is dead; say so and drop it.
- **Overtaken** — the codebase moved and the deferral no longer applies. Say
  that plainly rather than leaving it to be re-read every session.

Deferrals blocking each other are common and worth checking for explicitly: two
plans that each deferred the same consolidation "because the other one was
editing that file" are both unblocked the moment both are DONE, and neither
plan's own notes will say so.
```

**Verificar**: `grep -c '^### Harvest the deferrals'
skills/improve/references/closing-the-loop.md` → `1`, y que quedó dentro de la
sección de `reconcile` y antes del párrafo de cierre:

```sh
awk '/^## `reconcile`/{r=NR} /^### Harvest the deferrals/{h=NR} /^Finish with a short report/{f=NR} END{print (r<h && h<f) ? "OK" : "FAIL"}' skills/improve/references/closing-the-loop.md
```

→ `OK`.

### Paso 3: Probarlo contra los diferidos reales de este repo

El procedimiento nuevo tiene que encontrar algo real. Este repo tiene dos
diferidos mutuamente bloqueados en planes ya DONE — son el caso de prueba.

Ejecuta la cosecha manualmente, tal como la haría un `reconcile`:

```sh
grep -n 'Deferido a propósito\|\*\*Deferred:\*\*' plans/00*.md
```

Debe devolver al menos las tres líneas de `plans/001` y `plans/002`. Léelas y
confirma que el procedimiento del Paso 2 las clasifica sin ambigüedad: dos están
**desbloqueadas** (001 y 002 están DONE, y cada una esperaba a la otra) y una
—el chequeo automatizado de concordancia entre las tres copias de las Hard
Rules— sigue **bloqueada**, esperando la CI del candidato 009.

**No ejecutes la consolidación.** Solo comprueba que el procedimiento la
identifica, y **anota el resultado en tu reporte**: qué diferidos encontró, cómo
los clasificó, y cuál sería el trabajo ejecutable resultante.

**Verificar**: el grep devuelve ≥3 coincidencias, y tu reporte contiene la
clasificación de cada una.

## Plan de pruebas

Este repo no tiene suite de tests — es el candidato 008 del índice y está fuera
del alcance de este plan. No introduzcas un framework de testing.

La verificación tiene dos partes:

- **Textual** (pasos 1 y 2): los greps y el `awk` de posición.
- **Funcional** (paso 3): el procedimiento nuevo se ejecuta contra los diferidos
  reales de este repo y produce una clasificación correcta. Este es el chequeo
  con contenido real — un procedimiento que no encuentra los dos diferidos
  conocidos está mal escrito aunque los greps pasen.

Nota sobre el marcador: los diferidos existentes usan `**Deferido a propósito**`
(español, convención accidental) y el marcador nuevo es `**Deferred:**`. Es
deliberado que el Paso 3 busque ambos: los planes viejos no se reescriben, y el
procedimiento debe tolerar que el histórico no siga la convención nueva.
Menciónalo en tu reporte si te parece que merece una nota en la skill.

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `grep -c 'starting with `\*\*Deferred:\*\*`' skills/improve/references/plan-template.md` → `1`
- [ ] `grep -c '^```' skills/improve/references/plan-template.md` devuelve un número **par**
- [ ] `grep -c '^### Harvest the deferrals' skills/improve/references/closing-the-loop.md` → `1`
- [ ] El comando `awk` del Paso 2 imprime `OK`
- [ ] La rama DONE remite a la subsección: `grep -c 'see "Harvest the deferrals" below' skills/improve/references/closing-the-loop.md` → `1`
- [ ] Las cuatro ramas de estado siguen presentes: `grep -c '^- \*\*DONE\*\*\|^- \*\*BLOCKED\*\*\|^- \*\*IN PROGRESS\*\*\|^- \*\*TODO\*\*' skills/improve/references/closing-the-loop.md` → `4`
- [ ] Los diferidos históricos siguen intactos: `git diff --name-only main...HEAD -- plans/001-propagar-hard-rules-y-reordenar-review.md plans/002-comandos-declarados-vs-ejecutados.md` → sin salida
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas: `plans/README.md`, `skills/improve/references/closing-the-loop.md`, `skills/improve/references/plan-template.md`
- [ ] Tu reporte contiene la clasificación de los diferidos encontrados en el Paso 3
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- El procedimiento de `reconcile` o la sección de notas de mantenimiento no
  coinciden con los extractos de "Estado actual" — el repo derivó desde
  `9ac4b2e`.
- El conteo de triple-backtick de `plan-template.md` queda **impar** tras el
  Paso 1: rompiste el bloque de plantilla. Revierte ese paso y reporta.
- El grep del Paso 3 devuelve menos de 3 coincidencias: los diferidos históricos
  fueron editados o borrados, y el caso de prueba ya no existe. Reporta qué
  encontraste.
- Te encuentras a punto de ejecutar la consolidación que describen los diferidos
  de 001 y 002, o de reescribir sus notas para que usen el marcador nuevo. Ambas
  cosas están explícitamente fuera de alcance: detente.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Qué mirar en el review**: que el Paso 1 haya caído **dentro** del bloque de
  plantilla. Una viñeta escrita fuera no llega a ningún plan generado y el
  cambio queda decorativo — es el modo de fallo más probable, y el mismo que
  acechaba al plan 002.
- **Interacción futura**: el marcador `**Deferred:**` es ahora contrato entre la
  plantilla y `reconcile`. Cambiarlo en un lado sin el otro rompe la cosecha en
  silencio: la búsqueda simplemente no encuentra nada, que es indistinguible de
  "no había diferidos". La CI del candidato 009 debería verificar que ambos
  archivos usan la misma cadena.
- **Trabajo ejecutable que este plan deja identificado, no hecho**: la
  consolidación de la nota "Note on fresh worktrees…" de `closing-the-loop.md`
  con el Paso 0 de la plantilla. Es el diferido que 001 y 002 se pasaron
  mutuamente. Sale a la superficie en el Paso 3 y merece su propio plan.
- **Deferred:** un chequeo automatizado de que el marcador concuerda entre
  `plan-template.md` y `closing-the-loop.md`. Bloqueado por el candidato 009
  (CI estructural) — no hay infraestructura donde ponerlo todavía.
