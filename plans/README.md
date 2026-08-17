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
| 003 | Declarar cuándo NO usar la skill | P2 | S | MED | — | TODO |
| 004 | Off-by-one, marketplace, `plans/` versionado, remote `origin` | P2 | S | LOW | — | TODO |
| 005 | Sincronizar la copia instalada de la skill | P1 | S | LOW | 001-004 | TODO |

Valores de estado: TODO | IN PROGRESS | DONE | BLOCKED (con motivo en una línea) |
REJECTED (con justificación en una línea).

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

## Despacho

Los planes **001-004 son despachables con `execute`**: tocan solo archivos del
repo y son seguros en un worktree aislado.

El plan **005 NO se despacha** — escribe en `~/.claude/skills/`, fuera del repo
y fuera del worktree. Ejecútalo a mano. Está marcado así en su encabezado.

## Fase C — candidatos, todavía sin planificar

Si se confirma el fork público diferenciado, estos son los planes a escribir. El
orden importa: la CI primero, porque es lo que impide que las correcciones de
001-004 se vuelvan a romper en el próximo merge.

| # | Candidato | Por qué es fase C |
|---|---|---|
| 005 | CI estructural: frontmatter, manifiestos, links relativos, paridad de variantes, concordancia de versión | Existe como PR #10 en el upstream (12 jun, 280 líneas, sin mergear) — evaluar `cherry-pick` con atribución antes que reimplementar |
| 006 | Perfil de recon para repos de prompts/agentes | **Diferenciador**: nadie lo intentó upstream. La skill hoy no puede auditarse a sí misma sin improvisar |
| 007 | Suite de evals propia | **Diferenciador**: nadie lo intentó upstream. Es lo único que detecta regresiones de prompt — este audit `deep` se perdió dos contradicciones que solo aparecieron leyendo títulos de PRs ajenas |
| 008 | Política de versionado y release; bump coordinado `plugin.json` ↔ frontmatter | La versión lleva en `1.0.0` desde el inicio pese a cambios en reglas duras; una copia instalada de junio quedó obsoleta sin señal alguna. Depende de 005 (la CI verifica la concordancia) |
| 009 | Refuerzo mecánico de "nunca edita código" (hook `PreToolUse`) | Confianza MED: verificar primero el esquema de hooks. Defensa en profundidad para un solo host — el README promete portabilidad a cualquier host Agent Skills, así que **no** reemplaza la regla en prompt |
| 010 | Renombre y posicionamiento del fork | `plugin.json:name` y el `name:` del frontmatter son el namespace de invocación. Ya falló en la práctica: una copia de junio en `~/.claude/skills/improve` le ganó al repo auditado |

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
