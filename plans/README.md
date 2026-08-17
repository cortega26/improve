# Planes de implementación

Generados por la skill `improve` el 2026-08-17, contra el commit `03369ee`,
auditando **este mismo repo** (dogfooding). Ejecuta en el orden de abajo salvo
que las dependencias digan otra cosa. Cada executor: lee el plan completo antes
de empezar, respeta sus condiciones de STOP, y actualiza su fila al terminar.

**Contexto de la decisión.** El upstream `shadcn/improve` lleva dos meses sin
merges (último: 2026-06-15) con 12 PRs abiertas. La ruta elegida es **A primero**
—versión propia corregida para uso personal— y **probablemente C después**— fork
público diferenciado. Los planes 001-004 son la fase A completa. La fase C está
listada más abajo como candidatos sin planificar todavía.

## Orden de ejecución y estado

| Plan | Título | Prioridad | Esfuerzo | Riesgo | Depende de | Estado |
|------|--------|-----------|----------|--------|------------|--------|
| 001 | Hard Rules 4+6 al executor; leer el diff antes de ejecutarlo | P1 | S | LOW | — | DONE (mergeado en `main` en `16aa72a`) |
| 002 | Procedencia de comandos (`declared`/`executed`) + Paso 0 de bootstrap | P1 | S | LOW | — | DONE (mergeado en `main` en `c8b3085`) |
| 003 | Declarar cuándo NO usar la skill | P2 | S | MED | — | DONE (mergeado en `main` en `0d13667`) |
| 004 | Off-by-one, marketplace, `plans/` versionado, remote `origin` | P2 | S | LOW | — | DONE (mergeado en `main` en `83a7d9a`) |
| 005 | Sincronizar la copia instalada de la skill | P1 | S | LOW | 001-004 | DONE (manual, `~/.claude/skills/improve` → `~/.agents/skills/improve`) |
| 006 | Realinear el README con la skill tras la fase A | P1 | S | LOW | 001, 002 (DONE) | DONE (mergeado en `main` en `e8333c4`, fast-forward) |
| 007 | Fechar el ejemplo congelado contra su versión de plantilla | P2 | S | LOW | 002 (DONE) | DONE (mergeado en `main` en `a229b9b`, rebase + fast-forward — ver Notas) |
| 008 | `reconcile` cosecha los diferidos de las notas de mantenimiento | P2 | S | LOW | — | DONE (mergeado en `main` en `8753715`, rebase + fast-forward — ver Notas) |

Valores de estado: TODO | IN PROGRESS | DONE | BLOCKED (con motivo en una línea) |
REJECTED (con justificación en una línea).

### Verificación (reconcile del 2026-08-17, contra HEAD)

Los cinco planes se verificaron contra el árbol vivo, no contra el reporte del
executor: **todos los criterios de done se cumplen**. Además:

- **Alcance limpio en las cinco ejecuciones.** `git diff --stat 03369ee..HEAD`
  toca 10 archivos y todos caen dentro de la unión de las listas "En alcance".
  Ningún archivo fuera de alcance fue modificado.
- **Contenido revisado, no solo greps.** Se leyeron los diffs de
  `closing-the-loop.md`, `SKILL.md` y `plan-template.md`: las Hard Rules 4 y 6
  se copiaron verbatim (sin parafrasear), el Paso 0 y el chequeo de alcance
  quedaron **dentro** del bloque de plantilla y la barra de calidad **fuera**,
  que es lo correcto. Los fences quedaron balanceados y el frontmatter YAML
  válido.
- **`audit-playbook.md` está limpio**: es el único archivo de la skill que
  ningún plan tocó, y no arrastra la clase de afirmación que el plan 002 retiró.

## Fase B — hallazgos post-ejecución, ya ejecutados

Surgieron *de* la ejecución de la fase A; ninguno es un fallo de los planes.
Los tres (006-008) están **DONE y mergeados** en `main` — ver las notas de cada
uno más abajo para el detalle de revisión.

| Plan | Hallazgo | Evidencia |
|---|---|---|
| 006 | **El README derivó de la skill en cuatro puntos.** Los cinco planes de la fase A lo dejaron fuera de alcance por buenas razones locales, y nadie miró el efecto acumulado | `README.md:112` describe el orden de review **anterior** al plan 001; `README.md:102` y `:103` mantienen la promesa de comandos verificados que el plan 002 retiró de la plantilla; `README.md:118-121` lista 4 Hard Rules cuando `SKILL.md` tiene 6 |
| 007 | **El ejemplo congelado ya no coincide con lo que la skill produce.** `README.md:68` lo enlaza como muestra del output | `examples/001-…` no tiene `Provenance`, ni Paso 0, y usa el chequeo `git status` que el plan 002 declaró defectuoso. Resolución elegida: **fecharlo, no regenerarlo** — conserva la autenticidad de ser un run real |
| 008 | **Los diferidos de las notas de mantenimiento quedan huérfanos.** 001 difirió alinear la nota de worktrees fríos (bloqueado por 002); 002 difirió consolidar `closing-the-loop.md:50` con el Paso 0 (bloqueado por 001). Ambos ya están DONE, ambos diferidos están desbloqueados, y nada los rastrea | `plans/001:274`, `plans/002:349`. `closing-the-loop.md` documenta `reconcile` para DONE/BLOCKED/IN PROGRESS/TODO y **no** dice nada de cosechar diferidos — es un hueco de la skill, no solo de este repo |

El 008 es el más interesante de los tres: no arregla este repo, corrige un hueco
del `reconcile` de la skill, y encaja como diferenciador junto a los candidatos
de fase C.

**Trabajo identificado pero no planificado**: la consolidación de la nota "Note
on fresh worktrees…" de `closing-the-loop.md:50` con el Paso 0 de la plantilla —
el diferido que 001 y 002 se pasaron mutuamente. El plan 008 lo saca a la
superficie en su Paso 3 pero deliberadamente **no** lo ejecuta; merece su propio
plan tras el primer `reconcile` con la cosecha ya implementada.

## Notas de dependencias

Los planes 001-004 son independientes entre sí y pueden ejecutarse en cualquier
orden. Pero **001 y 002 primero** por prioridad (001 es de seguridad, 002
sostiene la promesa central de la skill), y **004 último** por higiene de merge:
es el único que toca `SKILL.md` en cuatro puntos distintos y colisiona con 002
(línea 32) y 003 (líneas 3 y 107-120) si corre antes que ellos.

**005 va al final y depende de los cuatro**, porque sincroniza la copia
instalada con el resultado. Sin él, la fase A corrige el repo y tus runs siguen
usando la skill de junio — que es exactamente lo que pasó durante este audit.

Advertencia de conflicto: 001 edita `closing-the-loop.md`; 002 y 004
deliberadamente **no** lo tocan y documentan lo suyo en otros archivos para
evitar el choque. Si 001 se abandona, revisa esa decisión — hay una consolidación
pendiente anotada en las notas de mantenimiento de 002.

## Nota sobre 006 — aprobado y mergeado

Revisado como corresponde en `execute`: alcance limpio (solo `README.md`),
diff completo leído y comparado contra la "forma objetivo" de cada paso del
plan —coincide palabra por palabra—, las dos Hard Rules nuevas fueron
**resumidas**, no copiadas literal de `SKILL.md` (confirmado contra el texto
fuente), y los 11 criterios de done verificables por el executor se
re-corrieron en el worktree y pasan los 11. Los dos criterios restantes del
plan (chequeo de dos archivos, fila del índice) estaban deliberadamente
saltados por el override de despacho — le tocan al revisor, no al executor.

`main` no se había movido desde que se cortó la rama, así que el merge fue
fast-forward puro (`e8333c4`) — sin merge commit. Worktree y rama
`advisor/006-readme-realignment` ya limpiados.

## Nota sobre 007 — el worktree se creó con base desactualizada

El worktree del executor de 007 se cortó contra `9ac4b2e` (main **antes** de
mergear 006), no contra el `main` vigente en el momento del despacho. Su diff
crudo contra `main` mostraba **revertir** las tres correcciones de 006 —no por
un error del executor, que siguió el plan al pie de la letra, sino porque
trabajaba sobre una base vieja.

Verificado que no había overlap real de líneas (007 toca `README.md:81`; 006
tocó `88`, `102-103`, `112` y `118-121`), así que se resolvió con
`git rebase main` dentro del worktree — sin conflictos — y se revisó el diff
otra vez contra `main` antes de aprobar. El merge final fue fast-forward
(`a229b9b`). Los 5 criterios de done propios de 007 y los 3 de 006 se
re-verificaron tras el rebase; todos pasan.

**Nota menor sin bloquear**: el encabezado nuevo de `examples/001-…md` dice
`references/plan-template.md`, que desde ese directorio no es una ruta
relativa válida (sería `../skills/improve/references/plan-template.md`). Es
texto dentro de código inline, no un link markdown, así que no rompe el
render — es una imprecisión de redacción que yo mismo especifiqué en el plan.
No amerita otro round de revisión; si molesta, es un ajuste de una línea.

## Nota sobre 008 — mismo problema de base, dos hallazgos reales, una falsa alarma

Mismo patrón que 007: el worktree se cortó contra `9ac4b2e`, sin los merges de
006 ni 007. Como 008 no toca `README.md` ni `examples/` —solo
`plan-template.md` y `closing-the-loop.md`, que 006/007 no tocaron—, el
`git rebase main` fue trivial y sin conflictos. Merge final fast-forward
(`8753715`).

El executor reportó tres cosas que ameritaban verificación propia antes de
aceptarlas:

- **Falsa alarma, ya descartada**: dijo que `plans/README.md` ya usaba el slot
  008 para el candidato "Política de versionado" y que no existía
  `plans/008-*.md`. Ambas afirmaciones eran ciertas **solo contra el commit
  `9ac4b2e`** que veía su worktree — mis archivos `006/007/008.md` y la
  renumeración de la fase C a 009-014 nunca se comitearon, así que el worktree
  heredó el índice viejo. Contra el estado real (`plans/README.md` tal como
  queda en este commit), no hay colisión: el candidato 009 **es** la CI
  estructural, tal como dice el texto del plan 008. La "corrección" del
  executor sobre "009 debería ser 005" está descartada — era él viendo la
  numeración vieja, no un error mío.
- **Hallazgo real, aceptado**: la cosecha tal como quedó escrita busca
  literalmente `**Deferred:**`. Los cinco diferidos que existen hoy en
  `plans/001` a `plans/004` usan la convención vieja en español
  (`**Deferido a propósito**`), así que un `reconcile` corrido hoy encontraría
  **cero** de los cinco. El plan 008 ya avisaba de esto en su "Plan de
  pruebas" y lo dejó fuera de alcance a propósito — la nota queda registrada
  aquí para no perderla.
- **Hallazgo real, menor**: el diferido de `plans/004:284` (no tocar
  `.gitignore`, decisión del usuario) no encaja en ninguna de las tres
  categorías de la cosecha (Unblocked / Still blocked / Overtaken) — no está
  bloqueado por nada, es una decisión pendiente. La plantilla ya contempla ese
  caso ("nothing, it just wasn't worth doing now") pero la sección de cosecha
  no tiene su categoría. Candidato a una cuarta categoría en un ajuste futuro,
  no bloqueante.

## Orden recomendado para la fase B

Los tres son independientes y de esfuerzo S. Sugerencia: **006 primero** (es el
escaparate y la deriva más visible), **007 después** (una línea, cierra la
contradicción del ejemplo), **008 al final** — es el único que cambia el
comportamiento de la skill en vez de su documentación, y conviene ejecutarlo
cuando 006 y 007 ya no compitan por `README.md`.

Aviso de colisión: **006 y 007 tocan ambos `README.md`**. Si corren en paralelo
chocan; el 007 solo edita la frase de la sección `## Example` y el 006 no la
toca, así que el merge es trivial, pero ejecútalos en serie de todos modos.

## Despacho

Los planes **001-004 y 006-008 son despachables con `execute`**: tocan solo
archivos del repo y son seguros en un worktree aislado.

El plan **005 NO se despacha** — escribe en el directorio de skills instaladas,
fuera del repo y fuera del worktree. Ejecútalo a mano. Está marcado así en su
encabezado.

Ruta real tras la ejecución: la instalación vive en `~/.agents/skills/improve`, y
`~/.claude/skills/improve` es un **symlink** hacia ella (`readlink -f` en ambas
devuelve la misma ruta). Es una sola copia con dos nombres, no dos copias
compitiendo — la condición de STOP nº3 del plan 005 no aplicaba. El texto del
plan 005 sigue citando `~/.claude/skills/` porque se escribió antes de conocer
esa disposición; es un plan DONE y queda como registro histórico.

## Fase C — candidatos, todavía sin planificar

Si se confirma el fork público diferenciado, estos son los planes a escribir. El
orden importa: la CI primero, porque es lo que impide que las correcciones de
001-004 se vuelvan a romper en el próximo merge.

Numeración: las fases A y B consumieron 001-008, así que los candidatos
arrancan en 009.

| # | Candidato | Por qué es fase C |
|---|---|---|
| 009 | CI estructural: frontmatter, manifiestos, links relativos, paridad de variantes, concordancia de versión | Existe como PR #10 en el upstream (12 jun, 280 líneas, sin mergear) — evaluar `cherry-pick` con atribución antes que reimplementar. **Tres planes ya dependen de ella**: 006 y 008 la nombran en sus notas de mantenimiento como el arreglo estructural de la deriva que parchean |
| 010 | Perfil de recon para repos de prompts/agentes | **Diferenciador**: nadie lo intentó upstream. La skill hoy no puede auditarse a sí misma sin improvisar |
| 011 | Suite de evals propia | **Diferenciador**: nadie lo intentó upstream. Es lo único que detecta regresiones de prompt — este audit `deep` se perdió dos contradicciones que solo aparecieron leyendo títulos de PRs ajenas |
| 012 | Política de versionado y release; bump coordinado `plugin.json` ↔ frontmatter | La versión lleva en `1.0.0` desde el inicio pese a cambios en reglas duras **y en la `description` del frontmatter**, que es el disparador de ruteo: una copia obsoleta no solo pierde correcciones, se **activa distinto**. Depende de 009 (la CI verifica la concordancia) |
| 013 | Refuerzo mecánico de "nunca edita código" (hook `PreToolUse`) | Confianza MED: verificar primero el esquema de hooks. Defensa en profundidad para un solo host — el README promete portabilidad a cualquier host Agent Skills, así que **no** reemplaza la regla en prompt |
| 014 | Renombre y posicionamiento del fork | `plugin.json:name` y el `name:` del frontmatter son el namespace de invocación. Ya falló en la práctica: una copia de junio en `~/.claude/skills/improve` le ganó al repo auditado |

Obligación de licencia para toda la fase C: `LICENSE.md` conserva `MIT © shadcn`.
Se **agrega** una línea de copyright propia, no se reemplaza la existente. Si se
hace `cherry-pick` de PRs abiertas del upstream (#8, #10, #20), verificar que la
autoría de los commits sobreviva al rebase.

## Hallazgos considerados y rechazados

Registrados para que no se re-auditen en el próximo run:

- **URLs de fork en `plugin.json`** (`homepage`/`repository` apuntan a
  `shadcn/improve`): correcto mientras el fork no se posicione como proyecto
  propio. Se revisa en el plan 010, no antes.
- **`plugin.json` sin campo `skills`**: los plugins autodescubren `skills/`. No es
  un defecto.
- **Falta `.gitignore`**: repo solo-markdown, sin artefactos de build. Costo cero.
- **`LICENSE.md` en vez de `LICENSE`**: GitHub lo reconoce igual. Cosmético.
- **PR #16 del upstream** ("pin execute worktree to the plan's base commit"):
  cerrada sin mergear, motivo desconocido. **No rechazada** — pendiente de
  investigar si describe un defecto real antes de decidir.

## Qué no se auditó

No hay código ejecutable en este repo, así que corrección, rendimiento y
dependencias se reducen a consistencia interna del prompt — eso sí se cubrió
completo (875 líneas, los 9 archivos). Lo que **no** se hizo: ejecutar la skill
contra repos de prueba reales para medir su comportamiento. Eso es exactamente
lo que propone el candidato 007.
