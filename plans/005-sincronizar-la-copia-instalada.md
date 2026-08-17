# Plan 005: Sincronizar la copia instalada de la skill con este repo

> ⚠️ **Este plan NO se despacha con `execute`.** Todos los demás planes tocan
> únicamente archivos del repo y son seguros en un worktree aislado. Este
> escribe en `~/.claude/skills/` — fuera del repo y fuera del worktree. Un
> executor aislado no puede hacerlo de forma significativa: o no tendría efecto,
> o modificaría el entorno real del usuario desde un sandbox que se supone
> desechable. **Ejecútalo tú, a mano, después de que 001-004 estén DONE.**

> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 03369ee..HEAD -- skills/`
> Aquí la deriva es lo *esperado*: deben aparecer los cambios de los planes
> 001-004. Si `skills/` no cambió en absoluto, este plan no tiene nada que
> sincronizar — detente.

## Estado

- **Prioridad**: P1
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: `plans/001`, `plans/002`, `plans/003`, `plans/004` (los cuatro
  en DONE)
- **Categoría**: dx
- **Planificado en**: commit `03369ee`, 2026-08-17

## Por qué importa

La condición de éxito de la fase A no es "el repo está corregido", es **"mis
runs usan la skill corregida"**. Esas son dos cosas distintas y hoy están
desconectadas.

La evidencia es de esta misma sesión. Al auditar este repo, la skill que se
cargó vino de `~/.claude/skills/improve`, una copia del 11 de junio de 2026. Le
faltaban la Hard Rule 6 (contenido del repo como dato, no instrucción), la
ingesta de docs de intención en Recon, el chequeo de visibilidad pública en
`--issues` y la resolución `plans/` vs `advisor-plans/`. Es decir: el repo llevaba
meses con correcciones que el usuario nunca recibió, y nada lo señalaba —
`plugin.json` y el frontmatter siguen diciendo `1.0.0` en ambos lados, así que
ni siquiera comparando versiones se detecta.

Sin este plan, los planes 001-004 aterrizan en el repo y la skill que tú
ejecutas sigue siendo la de junio.

## Estado actual

- `skills/improve/` en este repo — la fuente, con los cambios de 001-004
  aplicados.
- `~/.claude/skills/improve/` — la copia instalada, con fecha de modificación
  del 11 de junio de 2026. Contiene `SKILL.md` y `references/` con los tres
  archivos, misma estructura que el repo.

Divergencia confirmada hoy con
`diff /home/carlos/.claude/skills/improve/SKILL.md skills/improve/SKILL.md`:
seis bloques distintos, todos correcciones que existen en el repo y no en la
copia instalada.

## Comandos que vas a necesitar

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Ver la divergencia | `diff -r ~/.claude/skills/improve/ skills/improve/` | ejecutado | lista de diferencias antes; **sin salida** después |
| Fecha de la copia instalada | `ls -la ~/.claude/skills/improve/` | ejecutado | tras sincronizar, fecha de hoy |
| Inventario de archivos | `ls skills/improve/ skills/improve/references/` | ejecutado | `SKILL.md` + 3 referencias |

## Alcance

**En alcance**:
- `~/.claude/skills/improve/` (o la ruta donde tu host instale la skill)
- `plans/README.md` (fila de estado)

**Fuera de alcance**:
- Todo `skills/` de este repo — es la fuente; este plan la copia, no la edita.
  Si al sincronizar descubres que quieres cambiar algo, ese cambio va en un plan
  nuevo, no aquí.
- Cualquier otra skill instalada bajo `~/.claude/skills/`.
- La política de versionado — es el candidato 008 de la fase C. Este plan
  sincroniza una vez; no resuelve cómo evitar que vuelva a divergir.

## Pasos

### Paso 1: Identificar cómo está instalada

Determina la vía real antes de reinstalar, porque la copia y el gestor de
paquetes pueden pisarse:

```sh
ls -la ~/.claude/skills/improve/
ls -la ~/.claude/plugins/ 2>/dev/null | grep -i improve
```

Si aparece bajo `plugins/`, está instalada como plugin de Claude Code y debe
reinstalarse por esa vía. Si solo existe bajo `skills/`, es una instalación de
skill directa.

**Verificar**: sabes cuál de las dos vías aplica. Anótalo.

### Paso 2: Sincronizar

Según lo que determinaste:

- **Instalación de skill directa** — reinstala desde este repo con el mismo
  gestor que usaste originalmente (`npx skills add`, apuntando a tu fork o a la
  ruta local). Si el gestor no acepta rutas locales, copia el directorio:

  ```sh
  rm -rf ~/.claude/skills/improve
  cp -r skills/improve ~/.claude/skills/improve
  ```

- **Instalación como plugin** — reinstala por la vía de plugins de tu host,
  apuntando a tu fork.

**Verificar**: `diff -r ~/.claude/skills/improve/ skills/improve/` → **sin
salida**. Es el criterio central de este plan; cualquier salida significa que la
sincronización no fue completa.

### Paso 3: Confirmar que las correcciones llegaron

Comprueba en la copia **instalada** que los cambios de 001-004 están presentes:

```sh
grep -c 'never reproduce secret values' ~/.claude/skills/improve/references/closing-the-loop.md   # → 1  (plan 001)
grep -c 'Provenance' ~/.claude/skills/improve/references/plan-template.md                          # → ≥1 (plan 002)
grep -c 'When not to use this' ~/.claude/skills/improve/SKILL.md                                   # → 1  (plan 003)
grep -c '≤9 concurrent' ~/.claude/skills/improve/SKILL.md                                          # → 1  (plan 004)
grep -c '≤8' ~/.claude/skills/improve/SKILL.md                                                     # → 0  (plan 004)
```

**Verificar**: los cinco valores coinciden.

### Paso 4: Cerrar la sesión de la skill

La skill se carga al invocarse. Si tienes una sesión abierta que ya la cargó,
sigue usando la versión vieja en memoria. Abre una sesión nueva antes de confiar
en el resultado.

**Verificar**: en una sesión nueva, invoca la skill y confirma que el texto
cargado incluye la sección `## When not to use this`.

## Plan de pruebas

No aplica una suite: la verificación de este plan **es** el `diff -r` del Paso 2,
que es una comparación exacta byte a byte de los cuatro archivos. Los greps del
Paso 3 son redundantes por diseño — verifican que el `diff` limpio significa lo
que crees que significa, y no que ambos lados quedaron igualmente desactualizados.

## Criterios de done

- [ ] `diff -r ~/.claude/skills/improve/ skills/improve/` → sin salida
- [ ] Los cinco greps del Paso 3 dan los valores indicados
- [ ] `ls -la ~/.claude/skills/improve/SKILL.md` muestra fecha de hoy
- [ ] En una sesión nueva, la skill cargada contiene `## When not to use this`
- [ ] Fila de estado actualizada en `plans/README.md`

## Condiciones de STOP

Detente y reporta si:

- La copia instalada contiene cambios que **no** están en el repo. Significa que
  editaste la skill instalada directamente en algún momento y estarías a punto de
  perder ese trabajo con el `rm -rf`. Haz `diff` en ambas direcciones y porta esos
  cambios al repo primero, como un plan nuevo.
- `~/.claude/skills/improve/` no existe, o la skill se resuelve desde otra ruta
  que no identificaste en el Paso 1. Sincronizar la ruta equivocada deja el
  problema intacto y te hace creer que lo resolviste.
- La skill está instalada como plugin **y** como skill directa a la vez. Hay dos
  copias compitiendo y hay que decidir cuál gana antes de tocar nada — es el
  mismo problema de namespace que motiva el candidato 010.

## Notas de mantenimiento

- **Esto se va a volver a desincronizar.** Este plan es un arreglo puntual, no un
  mecanismo. La solución estructural es el candidato 008 (política de versionado):
  con un `version` que suba de verdad en cada cambio, comparar frontmatter contra
  `plugin.json` detecta la deriva sin hacer `diff -r` de todo el árbol.
- **Qué mirar**: que el `diff -r` esté limpio *en ambas direcciones*. `diff -r A B`
  ya reporta archivos que existen solo en uno de los dos lados, pero conviene leer
  la salida completa antes de declararla vacía en vez de confiar en el código de
  salida.
- **Si vas a la fase C** con un renombre (candidato 010), este plan se ejecuta una
  vez más al final, contra el nombre nuevo — y ahí sí conviene desinstalar la
  copia vieja explícitamente, para no quedar con dos skills compitiendo por el
  mismo disparador.
