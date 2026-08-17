# Plan 004: Cuatro correcciones de consistencia — off-by-one, instalación por marketplace, `plans/` versionado y remote `origin`

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 03369ee..HEAD -- skills/improve/SKILL.md README.md skills/improve/references/audit-playbook.md skills/improve/references/plan-template.md`
> Si alguno de esos archivos cambió desde que se escribió este plan, compara los
> extractos de "Estado actual" contra el código vivo antes de continuar; ante
> cualquier discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: ninguno (pero conviene ejecutarlo después de 001-003 para
  evitar conflictos de merge en `SKILL.md`)
- **Categoría**: bug + docs
- **Planificado en**: commit `03369ee`, 2026-08-17

## Por qué importa

Cuatro defectos independientes, todos pequeños, todos verificados, agrupados en
un plan porque ninguno justifica su propio ciclo de review y ninguno interactúa
con los otros:

1. **Off-by-one en el nivel `deep`.** La tabla de esfuerzo dice `≤8 concurrent,
   one per category` y, dos filas más abajo, `all nine`. El playbook tiene nueve
   categorías numeradas. En el modo más caro y exhaustivo de la skill, una
   categoría se queda sin subagente. Que la propia skill liste los off-by-one
   como categoría de corrección en el playbook §1 lo vuelve algo más que un
   typo.
2. **La instalación por marketplace es invisible.** El commit `e98afc5` agregó
   `.claude-plugin/marketplace.json` y el README nunca lo mencionó. La vía de
   instalación existe, funciona, y ningún usuario se entera.
3. **`plans/` debe estar commiteado y nadie lo dice.** `closing-the-loop.md:23`
   rodea el problema —inlinea el texto del plan porque *"the worktree contains
   only committed files"*— mientras el chequeo de deriva de la plantilla y todo
   el flujo `reconcile` asumen que los planes están versionados. La dependencia
   es real, está implícita, y su modo de fallo es silencioso: un usuario con
   `plans/` en `.gitignore` obtiene un `reconcile` que no encuentra nada.
4. **`branch` asume un remote llamado `origin`.** La variante hardcodea
   `origin/<default>`. Falla en clones sin `origin`, con el remote renombrado, o
   —el caso que motivó este audit— en un fork donde `origin` es el fork y
   `upstream` es el repo original.

## Estado actual

**1. Off-by-one** (`SKILL.md:57` y `SKILL.md:59`), filas de la tabla de esfuerzo:

```markdown
| Subagents | 0–1 (sweep directly when feasible) | ≤4 concurrent | ≤8 concurrent, one per category |
| Categories | correctness, security, tests | all nine | all nine |
```

El playbook tiene nueve secciones numeradas. Verificado con
`grep -c '^## [0-9]' skills/improve/references/audit-playbook.md` → `9`.

**2. Instalación** (`README.md:13-19`):

~~~markdown
## Install

```bash
npx skills add shadcn/improve
```

Works in any agent that supports [Agent Skills](https://agentskills.io) format. The plans it writes are plain markdown, so any agent (or human) can pick them up.
~~~

Y el manifiesto que existe pero no se documenta, `.claude-plugin/marketplace.json`:

```json
{
  "name": "improve",
  "owner": { "name": "shadcn" },
  "metadata": { "description": "Claude marketplace catalog for the improve plugin." },
  "plugins": [ { "name": "improve", "source": "./" } ]
}
```

**3. `plans/` versionado** (`closing-the-loop.md:23`), el workaround existente:

```markdown
1. **The full plan file text, inlined.** The worktree contains only committed files — if `plans/` is uncommitted, the executor can't read it. Never assume; always inline.
```

Y el supuesto contrario, en el chequeo de deriva de `plan-template.md:27`:

```markdown
> **Drift check (run first)**: `git diff --stat <planned-at SHA>..HEAD -- <in-scope paths>`
```

**4. Remote hardcodeado** (`SKILL.md:112`), variante `branch`:

```markdown
- `branch` → audit only the current working branch's changes: scope = files changed since the merge-base with the default branch (`git diff --name-only $(git merge-base origin/<default> HEAD)..HEAD`) plus their direct importers/callers.
```

**Convenciones del repo que debes respetar:**

- Repo **solo markdown**: sin build, sin tests, sin gestor de paquetes.
- El contenido de la skill y el README están **en inglés**. Escribe en inglés
  todo lo que agregues, aunque este plan esté en español.
- Estilo de commits: `fix: ...`, `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

Este repo no tiene build, tests, lint ni typecheck.

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 03369ee..HEAD -- <rutas en alcance>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -n '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Conteo de categorías | `grep -c '^## [0-9]' skills/improve/references/audit-playbook.md` | ejecutado en recon | `9` |
| Estado del árbol | `git status --porcelain` | ejecutado en recon | solo archivos en alcance |

## Alcance

**En alcance** (los únicos archivos que debes modificar):
- `skills/improve/SKILL.md` (pasos 1, 3 y 4)
- `README.md` (paso 2)

**Fuera de alcance** (NO tocar, aunque parezcan relacionados):
- `skills/improve/references/audit-playbook.md` — tiene nueve categorías y está
  **correcto**. El defecto está en la tabla que lo cuenta mal, no en él. No
  agregues ni fusiones categorías para que el número cuadre con el `8`.
- `.claude-plugin/marketplace.json` y `.claude-plugin/plugin.json` — el manifiesto
  funciona; lo que falta es documentarlo. Su contenido no cambia aquí.
- `skills/improve/references/closing-the-loop.md` — el plan 001 lo está
  editando; el paso 3 de este plan documenta el requisito en `SKILL.md` para
  evitar el conflicto.
- La numeración de las Hard Rules — el paso 3 agrega una frase a la Fase 4, no
  una regla nueva.

## Flujo de git

- Rama: `advisor/004-consistency-fixes`
- **Un commit por paso** — son cuatro defectos independientes y deben poder
  revertirse por separado. Mensajes en inglés siguiendo el estilo del repo:
  `fix: deep runs one subagent per category (9, not 8)`,
  `docs: document Claude Code marketplace installation`, etc.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 1: Corregir el off-by-one de subagentes

En `skills/improve/SKILL.md`, línea 57, cambia `≤8 concurrent, one per category`
por `≤9 concurrent, one per category`.

La fila resultante:

```markdown
| Subagents | 0–1 (sweep directly when feasible) | ≤4 concurrent | ≤9 concurrent, one per category |
```

**Verificar**: `grep -c '≤9 concurrent, one per category' skills/improve/SKILL.md`
→ `1`, y `grep -c '≤8' skills/improve/SKILL.md` → `0`. El número debe seguir
concordando con el conteo real:
`grep -c '^## [0-9]' skills/improve/references/audit-playbook.md` → `9`.

### Paso 2: Documentar la instalación por marketplace

En `README.md`, dentro de la sección `## Install`, agrega la segunda vía después
del bloque `npx skills add`, conservando el existente:

~~~markdown
Or, as a Claude Code plugin:

```bash
/plugin marketplace add shadcn/improve
/plugin install improve@improve
```
~~~

El delimitador exterior `~~~` es solo para anidar el ejemplo dentro de este
plan. Lo que escribes en `README.md` usa triple backtick, igual que el resto
del archivo.

Verifica los nombres antes de escribirlos: el del marketplace y el del plugin
salen de `.claude-plugin/marketplace.json` y `.claude-plugin/plugin.json`
respectivamente. Si no coinciden con `improve@improve`, usa los reales y anótalo
en tu reporte.

**Verificar**: `grep -c 'plugin marketplace add' README.md` → `1`, y
`grep -c 'npx skills add' README.md` → `1` (la vía original sigue ahí).

### Paso 3: Declarar que `plans/` debe estar versionado

En `skills/improve/SKILL.md`, al final de la Fase 4 (después del párrafo que
termina con la escritura de `plans/README.md`, alrededor de la línea 105),
agrega:

```markdown
**Tell the user to commit `plans/`.** Three downstream flows assume it is
versioned: `execute` can only hand the worktree committed files, every plan's
drift check diffs against its `Planned at` SHA, and `reconcile` reads the index
as the record of what happened. A gitignored `plans/` fails all three silently.
If the repo's ignore rules would exclude it, say so when you present the plans.
```

**Verificar**: `grep -c 'Tell the user to commit' skills/improve/SKILL.md` → `1`.

### Paso 4: Quitar el supuesto de que el remote se llama `origin`

En `skills/improve/SKILL.md`, línea 112, reemplaza el comando hardcodeado por
una resolución del remote. Forma objetivo del fragmento:

```markdown
- `branch` → audit only the current working branch's changes: scope = files changed since the merge-base with the default branch (resolve the base ref first — `git symbolic-ref --quiet refs/remotes/<remote>/HEAD` for whichever remote tracks this repo, falling back to the local default branch when there is no remote at all — then `git diff --name-only $(git merge-base <base-ref> HEAD)..HEAD`) plus their direct importers/callers. On a fork, confirm with the user whether the base is their fork's default branch or the upstream's before scoping; the two give very different answers.
```

**Verificar**: `grep -c 'origin/<default>' skills/improve/SKILL.md` → `0`, y
`grep -c 'On a fork, confirm with the user' skills/improve/SKILL.md` → `1`.

## Plan de pruebas

Este repo no tiene suite de tests — es el hallazgo 10 del audit y está fuera del
alcance de este plan. No introduzcas un framework de testing.

La verificación es textual, más un chequeo de consistencia cruzada que es el
único con contenido real: **el número de la tabla de subagentes debe seguir al
conteo de categorías del playbook**, no ser una constante escrita a mano. El
comando del Paso 1 verifica ambos lados. Este es exactamente el chequeo que la
CI de la fase C debe automatizar.

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `grep -c '≤9 concurrent, one per category' skills/improve/SKILL.md` → `1`
- [ ] `grep -c '≤8' skills/improve/SKILL.md` → `0`
- [ ] `grep -c '^## [0-9]' skills/improve/references/audit-playbook.md` → `9` (sin cambios)
- [ ] `grep -c 'plugin marketplace add' README.md` → `1`
- [ ] `grep -c 'npx skills add' README.md` → `1`
- [ ] `grep -c 'Tell the user to commit' skills/improve/SKILL.md` → `1`
- [ ] `grep -c 'origin/<default>' skills/improve/SKILL.md` → `0`
- [ ] `grep -c 'On a fork, confirm with the user' skills/improve/SKILL.md` → `1`
- [ ] Al menos cuatro commits en la rama, uno por paso: `git log --oneline main..HEAD | wc -l` devuelve `4` o más
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas, sin ninguna otra: `README.md`, `plans/README.md`, `skills/improve/SKILL.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada. La forma con tres
> puntos compara contra el merge-base, así que sigue siendo correcta aunque los
> planes 001-003 hayan aterrizado en `main` mientras trabajabas — que es el
> orden recomendado.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- Alguno de los cuatro extractos de "Estado actual" no coincide con el archivo
  vivo — el repo derivó desde `03369ee`.
- `grep -c '^## [0-9]' skills/improve/references/audit-playbook.md` devuelve algo
  distinto de `9`: el playbook cambió de categorías y el número correcto del
  Paso 1 ya no es `9`. Reporta el conteo real, no lo adivines.
- Los nombres reales de marketplace y plugin no permiten construir un comando de
  instalación válido (Paso 2). Documentar un comando que no funciona es peor que
  no documentarlo.
- Te encuentras a punto de editar `audit-playbook.md` para fusionar categorías.
  Eso invierte la dirección del arreglo: detente.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Interacción futura**: el `≤9` es ahora un número acoplado a mano al conteo de
  categorías del playbook. Si alguien agrega una décima categoría, la tabla
  vuelve a mentir. El plan de CI de la fase C debería derivar o verificar ese
  número automáticamente — es el ejemplo más barato de chequeo estructural y
  buen primer caso de prueba para esa CI.
- **Qué mirar en el review**: el Paso 4 cambia una instrucción operativa para un
  agente, no solo prosa. Léelo como si fueras el agente: ¿queda claro qué ref
  usar cuando hay dos remotes? Si no, es un fallo de redacción aunque el grep
  pase.
- **Deferido a propósito**: no se agregó `plans/` a ningún `.gitignore` ni se
  tocó la configuración de git de nadie. El paso 3 solo instruye al asesor a
  *decirlo*; la decisión sigue siendo del usuario.
