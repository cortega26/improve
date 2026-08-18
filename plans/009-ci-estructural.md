# Plan 009: CI estructural — cherry-pick del PR #10 del upstream

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 8455800..HEAD -- .github/ scripts/ skills/improve/SKILL.md .claude-plugin/plugin.json README.md`
> Si algo cambió desde que se escribió este plan, compara los extractos de
> "Estado actual" contra el código vivo antes de continuar; ante cualquier
> discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P1
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: ninguno
- **Categoría**: dx
- **Planificado en**: commit `8455800`, 2026-08-17

## Por qué importa

Este repo lleva tres commits correctivos en su historial arreglando
contradicciones que un chequeo mecánico habría atrapado en el PR
(`6f89ebf`, `c0accf0`, `bd0aff9`), y la fase B de este mismo ciclo de trabajo
repitió el patrón: el README derivó de la skill en cinco puntos sin que nada lo
señalara hasta una revisión manual. No hay nada hoy que verifique que
`SKILL.md` sigue teniendo frontmatter válido, que `plugin.json` sigue siendo
JSON parseable, que los links relativos entre `SKILL.md` y sus `references/`
siguen resolviendo, o que la versión declarada en ambos manifiestos concuerda.

**No hace falta escribirlo desde cero.** El upstream `shadcn/improve` tiene el
PR #10 abierto desde el 12 de junio de 2026 (autor `CnxLuc`, sin mergear, 280
líneas) que resuelve exactamente esto: un script Python de biblioteca estándar
(sin dependencias) más un workflow de GitHub Actions de 11 líneas. Verificado
en esta sesión — `git cherry-pick -x` de sus dos commits aplica limpio contra
nuestro `main` actual, y `python3 scripts/check.py` corre en verde contra el
estado presente del repo (frontmatter válido, manifiestos concuerdan, links
resuelven, los 9 variants aparecen en README y SKILL.md, versiones `1.0.0` en
ambos lados). El código ya está escrito, probado por su autor ("negative-tested:
renaming `references/audit-playbook.md` fails the run"), y no reinventa nada
que este plan tendría que re-especificar.

La obligación de licencia que rige toda la fase C aplica aquí en su forma más
simple: `git cherry-pick -x` preserva la autoría original de `CnxLuc` en cada
commit — no hay reimplementación, no hay cuestión de atribución que resolver
más allá de mantener esa autoría intacta.

## Estado actual

**Los dos commits del PR**, ya fetcheados en este repo como objetos accesibles
por SHA (no dependen de que la ref `refs/pull-10-head` siga existiendo — los
objetos están en la base de datos compartida de git, visible desde cualquier
worktree de este repo):

| SHA | Mensaje | Archivo que crea |
|---|---|---|
| `ccc87ce26c260d31461c94a484227276491d9324` | `dx: add scripts/check.py structural checker` | `scripts/check.py` (269 líneas) |
| `5d70e724bfef9e4e5554278287b3db56298034a9` | `dx: run structural checks in CI` | `.github/workflows/check.yml` (11 líneas) |

Ambos autoría `CnxLuc <luc.de.leyritz@gmail.com>`.

**Qué valida `scripts/check.py`** (leído en esta sesión, íntegro):

1. `skills/improve/SKILL.md` tiene frontmatter con `name:` y `description:` no
   vacíos, y `name` es exactamente `improve`.
2. `.claude-plugin/plugin.json` parsea como JSON válido; `name`/`version`
   presentes; `name` coincide con el de `SKILL.md`.
3. Todo link markdown relativo (`[texto](ruta)`) en cualquier `.md` del repo
   resuelve a un archivo existente — salta bloques de código cercados (para no
   validar los links de ejemplo dentro de `plan-template.md`) y el directorio
   `plans/`.
4. Los 9 invocation variants (`quick`, `deep`, `branch`, `next`, `plan`,
   `review-plan`, `execute`, `reconcile`, `--issues`) aparecen como substring
   tanto en `README.md` como en `SKILL.md`.
5. El campo `version` anidado en el frontmatter `metadata:` de `SKILL.md`
   coincide con `plugin.json`'s `version`.

**Workflow** (`.github/workflows/check.yml`): dispara en push a `main` y en
cada PR; un solo job que hace checkout y corre `python3 scripts/check.py`. Sin
instalación de dependencias — el script es stdlib puro.

**Estado presente de este repo** (verificado con un dry-run en un worktree
descartable, ya limpiado):

```
$ python3 scripts/check.py
PASS check1: SKILL.md frontmatter valid (name='improve', description present)
PASS check2: plugin.json valid, name='improve' matches SKILL.md
PASS check3: all relative links resolve
PASS check4: all 9 variants present in both README.md and SKILL.md
PASS check5: version agrees: '1.0.0'
all checks passed
```

**Convenciones del repo que debes respetar:**

- No hay build, tests, lint ni typecheck previos a este plan — este plan **es**
  el primero en agregar una superficie ejecutable (`.github/workflows/`,
  `scripts/`). No introduzcas nada además de lo que trae el PR.
- El contenido de `scripts/check.py` está en inglés (comentarios y mensajes);
  consérvalo así — es código cherry-picked, no lo traduzcas ni lo reformatees.
- Estilo de commits: los dos que llegan por cherry-pick conservan su mensaje
  original (`dx: ...`). No los reescribas.

## Comandos que vas a necesitar

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 8455800..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Cherry-pick | `git cherry-pick -x ccc87ce26c260d31461c94a484227276491d9324 5d70e724bfef9e4e5554278287b3db56298034a9` | ejecutado en recon (dry-run en worktree descartable) | dos commits aplicados, exit 0 |
| Checker | `python3 scripts/check.py` | ejecutado en recon | `all checks passed`, exit 0 |
| Versión de Python | `python3 --version` | ejecutado en recon | `Python 3.13.12` (cualquier Python 3 sirve — el script es stdlib puro) |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | ver criterios de done |

## Alcance

**En alcance** (los únicos archivos que este plan crea):
- `scripts/check.py` (vía cherry-pick, contenido fijo — no lo edites)
- `.github/workflows/check.yml` (vía cherry-pick, contenido fijo — no lo edites)

**Fuera de alcance** (NO tocar, aunque parezcan relacionados):
- **Cualquier archivo existente.** Los dos commits del PR #10 solo *crean*
  archivos nuevos; no tocan `SKILL.md`, `README.md`, `plugin.json`, ni nada de
  `references/`. Si el cherry-pick te pide resolver un conflicto en un archivo
  existente, es una condición de STOP — no la resuelvas.
- **El contenido de `scripts/check.py` o `check.yml`.** Llegan completos y
  probados por su autor. No agregues checks, no cambies el trigger del
  workflow, no lo "mejores". Si ves algo que te parece mejorable, anótalo en tu
  reporte para que el revisor decida — no lo cambies tú.
- Los candidatos 012 (versionado) y 013 (hook `PreToolUse`) que dependen de
  esta CI — son planes separados, no parte de este.

## Flujo de git

- Rama: `advisor/009-structural-ci`
- Cherry-pick preserva los dos commits originales tal cual (con `-x`, que
  agrega una línea `(cherry picked from commit ...)` al mensaje). No hagas un
  commit propio adicional salvo que el Paso 2 te lo pida explícitamente.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 0: Establecer línea base

Este repo no tiene build/test/lint previos a este plan, así que no hay un
Paso 0 de "confirmar que ya estaba verde" en el sentido de la plantilla — el
propio plan *introduce* la primera superficie verificable. En su lugar,
confirma que el punto de partida está limpio:

```sh
git status --porcelain   # → sin salida (árbol limpio antes de empezar)
git cat-file -t ccc87ce26c260d31461c94a484227276491d9324   # → commit
git cat-file -t 5d70e724bfef9e4e5554278287b3db56298034a9   # → commit
```

Si cualquiera de los dos `cat-file` falla (objeto no encontrado), los commits
del PR no están disponibles en este worktree. Intenta
`git fetch https://github.com/shadcn/improve.git refs/pull/10/head` (operación
de red de solo lectura, no instala nada ni muta tu árbol de trabajo) y
reintenta. Si sigue fallando, es una condición de STOP.

**Verificar**: los tres comandos dan el resultado esperado.

### Paso 1: Cherry-pick de los dos commits

```sh
git cherry-pick -x ccc87ce26c260d31461c94a484227276491d9324 5d70e724bfef9e4e5554278287b3db56298034a9
```

Debe aplicar limpio, sin conflictos — ya fue verificado en esta sesión contra
el mismo punto de partida (`8455800`). Si el chequeo de deriva del encabezado
detectó cambios en `SKILL.md`, `plugin.json` o `README.md` desde entonces, el
cherry-pick sigue debiendo aplicar limpio porque el PR solo *crea* archivos
nuevos — pero si aun así hay conflicto, es una condición de STOP explícita, no
algo para resolver a mano.

**Verificar**: `git log --oneline -2` muestra los dos commits, cada uno con
`(cherry picked from commit ...)` en el cuerpo:
`git log -2 --format='%B' | grep -c 'cherry picked from commit'` → `2`.

### Paso 2: Confirmar que el checker corre en verde

```sh
python3 scripts/check.py
echo "exit: $?"
```

**Verificar**: la salida termina en `all checks passed` y el exit code es `0`.
Si algo falla, lee cuál de los cinco checks falló — probablemente significa que
el repo derivó de un modo que el checker correctamente detecta (por ejemplo, un
link relativo roto real). **No parchees el checker para que pase** — si el
fallo es real, repórtalo como hallazgo; si crees que es un falso positivo del
checker mismo, repórtalo igual y deja que el revisor decida.

## Plan de pruebas

El propio `scripts/check.py` **es** la suite de verificación que este plan
instala — no hay una suite separada que escribir. La prueba de que funciona es
que corre en verde contra el estado actual del repo (Paso 2) y que el workflow
de CI (`check.yml`) queda commiteado listo para disparar en el próximo push o
PR — aunque este plan, por Hard Rule 2, no puede simular ese disparo (correr
GitHub Actions localmente requeriría instalar `act` u otra herramienta, fuera
de alcance).

Verificación negativa opcional (no bloqueante, infórmalo si la corrés): el
autor del PR probó que renombrar `references/audit-playbook.md` hace fallar el
check 3 con el link roto nombrado. Podés reproducirlo en tu worktree para
confirmar que el checker detecta lo que dice detectar, y luego revertir el
renombre antes de continuar — pero no es un criterio de done, es una
verificación de confianza adicional.

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `test -f scripts/check.py && echo OK` → `OK`
- [ ] `test -f .github/workflows/check.yml && echo OK` → `OK`
- [ ] `git log -2 --format='%B' | grep -c 'cherry picked from commit'` → `2`
- [ ] `git log -2 --format='%an'` → ambas líneas dicen `CnxLuc` (autoría preservada)
- [ ] `python3 scripts/check.py` → termina en `all checks passed`, exit code `0`
- [ ] `git diff --name-only main...HEAD` → exactamente estas dos rutas: `.github/workflows/check.yml`, `scripts/check.py`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: a diferencia de los planes 001-008, este
> **no** te pide commitear plan-por-paso porque el cherry-pick ya produce los
> commits atómicos originales. `git diff --name-only main...HEAD` sigue siendo
> el chequeo correcto porque compara contra el merge-base, no contra un estado
> local que pueda haber cambiado.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- El cherry-pick produce un conflicto en cualquier archivo. Los commits del
  PR solo crean archivos nuevos — un conflicto significa que algo inesperado
  cambió, y resolverlo a mano arriesgaría alterar código que no escribiste ni
  revisaste.
- `python3 scripts/check.py` falla después de un cherry-pick limpio. Reporta
  cuál check falló y la línea exacta que señala — no lo arregles vos.
- Los objetos `ccc87ce...` o `5d70e72...` no existen ni tras el fetch de
  respaldo del Paso 0.
- Te encuentras a punto de editar el contenido de `check.py` o `check.yml`
  "para mejorarlos". Eso es exactamente lo que este plan decidió no hacer —
  llegan completos y probados; cualquier mejora es un plan aparte.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **Esto es lo que candidatos 012 y 001-008 estaban esperando.** El diferido
  registrado en `plans/001:271-273` (chequeo automatizado de concordancia
  entre las tres copias de las Hard Rules) y el candidato 012 (versionado)
  dependen de que esta CI exista primero — ahora existe. `reconcile` debería
  poder desbloquear ambos en su próxima corrida (usando la cosecha de
  diferidos del plan 008).
- **Qué mirar en el review**: que ambos commits conserven su SHA de origen en
  el mensaje (`cherry picked from commit ccc87ce...` / `5d70e72...`) y que
  `git log --format='%an'` siga diciendo `CnxLuc`, no el nombre de quien
  ejecuta este plan. Es la obligación de atribución de la fase C, y es
  mecánicamente verificable — no confíes solo en el reporte del executor.
- **Deferred:** el checker no valida nada sobre `plans/` en sí (lo excluye
  deliberadamente, según su propio código: `dirnames[:] = [d for d in
  dirnames if d not in (".git", "plans")...]`). Si en el futuro se quiere
  verificar la concordancia del marcador `**Deferred:**` entre
  `plan-template.md` y `closing-the-loop.md` (la nota de mantenimiento que
  dejó el plan 008), ese chequeo tendría que agregarse como un check nuevo —
  no bloqueado por nada, pero fuera de alcance de este plan porque el PR
  original no lo incluye y agregarlo sería modificar código que llegó
  completo.
