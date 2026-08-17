# Plan 007: Fechar el ejemplo congelado contra la versión de plantilla que lo produjo

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 9ac4b2e..HEAD -- examples/001-extract-shadow-config-resolution.md skills/improve/references/plan-template.md`
> Si alguno cambió desde que se escribió este plan, compara los extractos de
> "Estado actual" contra el código vivo antes de continuar; ante cualquier
> discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: `plans/002` (DONE — es su cambio el que dejó el ejemplo
  desalineado)
- **Categoría**: docs
- **Planificado en**: commit `9ac4b2e`, 2026-08-17

## Por qué importa

`examples/001-extract-shadow-config-resolution.md` es el escaparate del repo: el
README lo enlaza como la muestra de lo que la skill produce, y es lo único que
alguien evaluando el proyecto puede leer para juzgar la calidad del output sin
ejecutar nada.

El plan 002 cambió la plantilla en tres puntos, y el ejemplo no los tiene:

- no lleva la columna `Provenance` en su tabla de comandos,
- no lleva el Paso 0 de línea base verde,
- y usa como criterio de done el chequeo `git status` que el propio plan 002
  acaba de declarar defectuoso (pasa siempre después de commitear).

Dejarlo fuera de alcance durante la fase A fue lo correcto: reescribir un
artefacto congelado a mitad de una ejecución habría destruido lo que lo hace
creíble — ser un run real contra un repo real, no una muestra fabricada. Pero el
resultado es que el escaparate exhibe un formato que la skill ya no emite, y un
lector atento concluirá que la plantilla y el ejemplo se contradicen.

**La corrección elegida es fecharlo, no regenerarlo.** Regenerarlo obligaría a
inventar contenido que ningún run produjo (¿qué comandos habría marcado
`executed` el asesor en junio?), y eso convierte una muestra auténtica en una
simulación. Una línea de encabezado resuelve la contradicción y conserva la
autenticidad.

## Estado actual

Archivo a corregir: `examples/001-extract-shadow-config-resolution.md`.

**Encabezado actual** (líneas 1-4), que ya establece el patrón de "esto está
congelado" y es el lugar natural donde ampliarlo:

```markdown
> **Sample output.** A real plan produced by `/improve` against
> [shadcn/ui](https://github.com/shadcn-ui/ui) at commit `1994caba0`
> (2026-06-10), kept here as an example of the format. The codebase has
> moved on — don't execute this; run `/improve` on your own repo instead.
```

**Las tres divergencias**, verificadas:

| Elemento de la plantilla vigente | En el ejemplo |
|---|---|
| Columna `Provenance` en la tabla de comandos | ausente (`grep -c 'Provenance'` → `0`) |
| `### Step 0: Establish a green baseline` | ausente (`grep -c 'Step 0'` → `0`) |
| Chequeo de alcance por `git diff --name-only <base>...HEAD` | usa el viejo `No files outside the in-scope list are modified (git status)` (`grep -c` → `1`) |

**Enlace desde el README** (`README.md:68`, sección `## Example`):

```markdown
Picking #1 produced [this plan](./examples/001-extract-shadow-config-resolution.md) — current code excerpted, exact steps, the repo's own test/lint commands as verification gates, and STOP conditions for when reality doesn't match.
```

**Convenciones del repo que debes respetar:**

- Repo **solo markdown**: sin build, sin tests, sin gestor de paquetes.
- El contenido está **en inglés**. Escribe en inglés todo lo que agregues,
  aunque este plan esté en español.
- El encabezado del ejemplo usa blockquote (`> `). Mantén ese formato.
- Estilo de commits: `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

Este repo no tiene build, tests, lint ni typecheck.

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 9ac4b2e..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -c '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | ver criterios de done |

## Alcance

**En alcance**:
- `examples/001-extract-shadow-config-resolution.md` (solo el encabezado)
- `README.md` (una frase en la sección `## Example`)

**Fuera de alcance** (NO tocar, aunque parezca lo obvio):
- **El cuerpo del ejemplo**: pasos, tabla de comandos, criterios de done,
  condiciones de STOP. **No agregues `Provenance`, ni el Paso 0, ni cambies el
  chequeo `git status`.** Ese contenido es el registro de lo que la skill
  produjo el 2026-06-10 y su valor está en ser auténtico. Modificarlo es
  exactamente lo que este plan decidió no hacer.
- `skills/improve/references/plan-template.md` — la plantilla está correcta; es
  el ejemplo el que quedó atrás.
- Las otras derivas del README — las cubre el plan 006.

## Flujo de git

- Rama: `advisor/007-date-the-frozen-example`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `docs: date the frozen example against its template version`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 1: Ampliar el encabezado del ejemplo

En `examples/001-extract-shadow-config-resolution.md`, reemplaza el blockquote
de las líneas 1-4 por esta versión, que conserva el texto existente y añade la
salvedad de formato:

```markdown
> **Sample output.** A real plan produced by `/improve` against
> [shadcn/ui](https://github.com/shadcn-ui/ui) at commit `1994caba0`
> (2026-06-10), kept here as an example of the format. The codebase has
> moved on — don't execute this; run `/improve` on your own repo instead.
>
> **Written against the template as it stood in June 2026.** The current
> template adds two things this plan predates: a `Provenance` column marking
> each command as `declared` or `executed`, and a Step 0 that confirms the
> baseline is already green before any change. Its scope check also now uses
> `git diff --name-only <base>...HEAD` rather than `git status`, which reports
> nothing once the executor has committed. This file is kept unedited as the
> record of a real run — see `references/plan-template.md` for what the skill
> emits today.
```

**Verificar**: `grep -c 'Written against the template as it stood in June 2026'
examples/001-extract-shadow-config-resolution.md` → `1`, y que el cuerpo sigue
intacto — los dos elementos de la plantilla nueva no deben existir como
estructura, solo mencionarse en la prosa del encabezado:

```sh
grep -c '^### Step 0' examples/001-extract-shadow-config-resolution.md    # → 0
grep -c '| Provenance |' examples/001-extract-shadow-config-resolution.md # → 0
```

(No cuentes las apariciones sueltas de `Provenance` o `Step 0`: el encabezado
nuevo nombra ambas en líneas distintas, así que ese conteo es `2` y no dice
nada útil. Los dos greps de arriba son los que importan.)

### Paso 2: Ajustar la frase del README que lo presenta

En `README.md`, sección `## Example`, la frase que enlaza el ejemplo afirma que
muestra "the repo's own test/lint commands as verification gates" — cierto, pero
ahora incompleto. Reemplázala por:

```markdown
Picking #1 produced [this plan](./examples/001-extract-shadow-config-resolution.md) — current code excerpted, exact steps, the repo's own test/lint commands as verification gates, and STOP conditions for when reality doesn't match. It's kept unedited from a June 2026 run, so its header notes the two things the current template adds.
```

**Verificar**: `grep -c "kept unedited from a June 2026 run" README.md` → `1`.

## Plan de pruebas

Este repo no tiene suite de tests — es el candidato 008 del índice y está fuera
del alcance de este plan. No introduzcas un framework de testing.

La verificación es textual, y el chequeo con contenido real es **negativo**:
confirmar que el cuerpo del ejemplo *no* cambió. Compruébalo con git, que es más
fiable que cualquier grep:

```sh
git diff main...HEAD -- examples/001-extract-shadow-config-resolution.md | grep -c '^[+-][^+-]'
```

El patrón `^[+-][^+-]` excluye las cabeceras `---`/`+++` del diff, así que el
conteo es solo de líneas de contenido: debe corresponder al encabezado y nada
más (≈14 entre añadidas y borradas). Si supera 30, tocaste el cuerpo: lee el
diff completo antes de continuar.

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `grep -c 'Written against the template as it stood in June 2026' examples/001-extract-shadow-config-resolution.md` → `1`
- [ ] `grep -c '^### Step 0' examples/001-extract-shadow-config-resolution.md` → `0`
- [ ] `grep -c '| Provenance |' examples/001-extract-shadow-config-resolution.md` → `0`
- [ ] El cuerpo sigue intacto: `git diff main...HEAD -- examples/001-extract-shadow-config-resolution.md | grep -c '^[+-][^+-]'` devuelve menos de `30`
- [ ] El criterio viejo sigue en su sitio (es el registro histórico): `grep -c 'No files outside the in-scope list are modified' examples/001-extract-shadow-config-resolution.md` → `1`
- [ ] `grep -c "kept unedited from a June 2026 run" README.md` → `1`
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas: `README.md`, `examples/001-extract-shadow-config-resolution.md`, `plans/README.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- El encabezado del ejemplo no coincide con el extracto de "Estado actual" — el
  archivo derivó desde `9ac4b2e`.
- Los conteos de divergencia no son los esperados (`Provenance` → `0`,
  `Step 0` → `0`, chequeo `git status` → `1`) **antes** de tu cambio: significa
  que alguien ya tocó el ejemplo y la premisa del plan cambió.
- Te encuentras a punto de editar el cuerpo del ejemplo para "arreglarlo". Esa
  es explícitamente la decisión contraria a la que este plan tomó: detente y
  reporta.
- La plantilla vigente ya no tiene `Provenance` o el Paso 0: el encabezado que
  este plan escribe estaría describiendo una diferencia inexistente.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Este encabezado caduca.** Cada cambio futuro a la plantilla amplía la brecha
  con el ejemplo, y la nota tendrá que crecer o el ejemplo tendrá que
  regenerarse. Regla práctica: cuando la lista de diferencias pase de tres,
  regenerar sale más barato que seguir anotando — y para entonces conviene
  producirlo con un run real nuevo, no editando este a mano.
- **Qué mirar en el review**: que el cuerpo esté byte-idéntico. El valor de este
  archivo es la autenticidad; un ejemplo "arreglado" para verse moderno no
  demuestra nada sobre lo que la skill produce.
- **Deferred:** regenerar el ejemplo con un run real de `/improve` contra un
  repo público actual. Es la solución de fondo — muestra el formato vigente y
  sigue siendo auténtica — pero cuesta un run completo y una decisión sobre qué
  repo usar. Bloqueado por el candidato 011 (suite de evals), que necesitará
  repos de prueba de todas formas; reconsiderar cuando exista.
