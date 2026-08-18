# Plan 012: Política de versionado — bump coordinado y guía para futuros cambios

> **Instrucciones para el executor**: sigue este plan paso a paso. Ejecuta cada
> comando de verificación y confirma el resultado esperado antes de pasar al
> siguiente paso. Si ocurre algo listado en "Condiciones de STOP", detente y
> reporta — no improvises. Al terminar, actualiza la fila de estado de este plan
> en `plans/README.md`, salvo que un revisor te haya despachado y te haya dicho
> que él mantiene el índice.
>
> **Chequeo de deriva (ejecútalo primero)**:
> `git diff --stat 281903a..HEAD -- .claude-plugin/plugin.json skills/improve/SKILL.md README.md`
> Si algo cambió desde que se escribió este plan, compara los extractos de
> "Estado actual" contra el código vivo antes de continuar; ante cualquier
> discrepancia, trátalo como condición de STOP.

## Estado

- **Prioridad**: P2
- **Esfuerzo**: S
- **Riesgo**: LOW
- **Depende de**: `plans/009` (DONE — el chequeo de concordancia de versión ya
  existe en `scripts/check.py` check5; este plan lo usa como mecanismo de
  verificación, no lo reimplementa)
- **Categoría**: dx
- **Planificado en**: commit `281903a`, 2026-08-17

## Por qué importa

`plugin.json` y el frontmatter de `SKILL.md` declaran `"1.0.0"` desde el
primer commit de este repo. Desde entonces aterrizaron once planes que
cambiaron comportamiento real de la skill — no solo prosa cosmética: las Hard
Rules 4 y 6 se propagaron al executor (antes no llegaban), el orden del
review de `execute` se invirtió, se agregó una sección completa de "cuándo NO
usar esta skill" que hace que la `description` del frontmatter —el
disparador de ruteo— rechace explícitamente pedidos que antes podría haber
aceptado. Ese último punto es el que más importa: **una copia instalada
desactualizada no solo pierde correcciones, se activa distinto** — es
exactamente lo que le pasó a la copia de `~/.claude/skills/improve` que
sombreó este mismo repo durante el audit original de esta sesión.

Sin un número de versión que se mueva, no hay forma de que un usuario, un
host de skills, o un futuro `reconcile` sepan que algo cambió. El plan 009 ya
construyó el mecanismo de *verificación* (check5 de `scripts/check.py`
confirma que `plugin.json` y el frontmatter concuerdan) — lo que falta es
usarlo: mover el número a algo que refleje los cambios reales, y dejar
escrita la regla de cuándo mover qué segmento, para que el próximo cambio no
tenga que redescubrir el criterio.

## Estado actual

**`plugin.json`** (`.claude-plugin/plugin.json`, íntegro):

```json
{
  "name": "improve",
  "description": "Point it at a codebase and it figures out what's worth doing — bugs, perf, tech debt, what to build next — then writes plans any agent can execute. It can hand the work to a cheaper model and review the result. It never edits your code.",
  "version": "1.0.0",
  "author": {
    "name": "shadcn",
    "url": "https://twitter.com/shadcn"
  },
  "homepage": "https://github.com/shadcn/improve",
  "repository": "https://github.com/shadcn/improve",
  "license": "MIT",
  "keywords": ["audit", "code-review", "planning", "agents", "tech-debt", "security"]
}
```

**Frontmatter de `SKILL.md`** (`skills/improve/SKILL.md:1-8`):

```yaml
---
name: improve
description: Survey any codebase as a senior advisor...
license: MIT
metadata:
  author: shadcn
  version: "1.0.0"
---
```

**El mecanismo de verificación que ya existe** (`scripts/check.py`, check 5,
del plan 009 — no lo toques, solo úsalo):

```python
if plugin is not None and _skill_meta_version is not None:
    plugin_ver = plugin.get("version", "").strip()
    if _skill_meta_version != plugin_ver:
        fail(...)
    else:
        ok(f"check5: version agrees: {_skill_meta_version!r}")
```

**El criterio de bump para este cambio puntual**: desde `1.0.0` hasta ahora
aterrizaron cambios de comportamiento real (propagación de Hard Rules al
executor, orden del review invertido, sección nueva de alcance que cambia
qué pedidos acepta la skill) mezclados con correcciones y adiciones
aditivas (CI, perfil de recon, esqueleto de evals). Ninguno rompe una forma
de invocación existente ni quita una Hard Rule — son aditivos o correctivos.
Bajo semver aplicado a una skill (no a una API con contrato formal), esto
corresponde a un bump de **MINOR**: `1.0.0` → `1.1.0`.

**Convenciones del repo que debes respetar:**

- Repo **solo markdown** (más el script y workflow de 009, y el esqueleto de
  evals de 011 si ya aterrizó). No introduzcas dependencias nuevas.
- El contenido de la skill y del README está **en inglés**. Escribe en
  inglés todo lo que agregues, aunque este plan esté en español.
- Estilo de commits: `chore: ...`, `docs: ...` en inglés imperativo.

## Comandos que vas a necesitar

| Propósito | Comando | Procedencia | Resultado esperado |
|---|---|---|---|
| Chequeo de deriva | `git diff --stat 281903a..HEAD -- <rutas>` | ejecutado en recon | sin salida |
| Verificación de contenido | `grep -c '<patrón>' <archivo>` | ejecutado en recon | ver cada paso |
| Concordancia de versión | `python3 scripts/check.py` | ejecutado en recon | `all checks passed` (incluye check5) |
| Alcance | `git diff --name-only main...HEAD` | ejecutado en recon | ver criterios de done |

## Alcance

**En alcance**:
- `.claude-plugin/plugin.json` (solo el campo `version`)
- `skills/improve/SKILL.md` (solo la línea `version:` del frontmatter)
- `README.md` (una sección nueva y corta, "Versioning")

**Fuera de alcance** (NO tocar, aunque parezca relacionado):
- **`scripts/check.py`.** El chequeo de concordancia (check5) ya existe y
  funciona; este plan lo usa, no lo modifica.
- **Cualquier otro campo de `plugin.json` o del frontmatter** — `name`,
  `description`, `author`, etc. Este plan mueve un número, no reescribe
  metadata.
- **Un mecanismo de bump automatizado** (git hook, script de release). Fuera
  de alcance — este plan es la política en prosa más el bump puntual, no
  tooling nuevo. Si en el futuro se quiere automatizar, es un plan aparte.

## Flujo de git

- Rama: `advisor/012-versioning-policy`
- Un commit por paso. Mensajes en inglés siguiendo el estilo del repo, por
  ejemplo `chore: bump version to 1.1.0`.
- **No** hagas push ni abras PR salvo que el operador lo haya pedido.

## Pasos

### Paso 0: Confirmar la línea base

```sh
python3 scripts/check.py
```

Debe terminar en `all checks passed`, incluyendo `check5: version agrees:
'1.0.0'` — confirma que ambos archivos concuerdan *antes* de tu cambio, así
que el bump no está corrigiendo una discrepancia preexistente, solo
avanzando el número.

**Verificar**: la salida contiene `check5: version agrees: '1.0.0'`.

### Paso 1: Bump de `plugin.json`

En `.claude-plugin/plugin.json`, cambia únicamente:

```diff
-  "version": "1.0.0",
+  "version": "1.1.0",
```

**Verificar**: `grep -c '"version": "1.1.0"' .claude-plugin/plugin.json` →
`1`.

### Paso 2: Bump del frontmatter de `SKILL.md`

En `skills/improve/SKILL.md`, dentro del bloque `metadata:` del frontmatter,
cambia únicamente:

```diff
-  version: "1.0.0"
+  version: "1.1.0"
```

**Verificar**: `grep -c 'version: "1.1.0"' skills/improve/SKILL.md` → `1`; y
que la concordancia sigue siendo válida:

```sh
python3 scripts/check.py
```

Debe terminar en `all checks passed`, con `check5: version agrees: '1.1.0'`
(no `'1.0.0'` — confirmá que el número cambió en el mensaje, no solo que pasó).

### Paso 3: Documentar la política

En `README.md`, agrega una sección nueva `## Versioning` **entre** la sección
`## Hard rules` y `## License` (en ese orden — es donde vive la prosa de
norma del repo, justo antes del cierre). Forma objetivo:

```markdown
## Versioning

`plugin.json`'s `version` and `SKILL.md`'s frontmatter `metadata.version`
must move together — `scripts/check.py` (wired into CI) fails the build if
they drift, so there's no way to bump one and forget the other without CI
catching it.

What size bump, in order of how much it should worry an installed copy:

- **PATCH** — docs fixes, typo corrections, internal consistency repairs
  (a broken link, an off-by-one in a stated count) that don't change what
  the skill does or accepts.
- **MINOR** — additive changes: a new invocation variant, a new Hard Rule
  that only adds a restriction, a scoping clarification (like "when not to
  use this") that narrows what the skill accepts without removing an
  existing capability, new supporting infrastructure (CI, eval cases).
- **MAJOR** — anything that changes behavior for someone already using the
  skill: renaming or removing an invocation variant, relaxing or removing a
  Hard Rule, changing what a plan's Hard Rules require of an executor in a
  way that breaks a plan written against the old rules.

The distinction that matters most for this specific skill: `description` is
the routing trigger a host agent reads to decide whether to invoke `/improve`
at all. A change to it — even a "small" wording change — can change which
requests reach the skill. Treat any `description` change as at least MINOR,
even if the accompanying code change looks cosmetic.
```

**Verificar**: `grep -c '^## Versioning' README.md` → `1`; posición correcta:

```sh
awk '/^## Hard rules/{h=NR} /^## Versioning/{v=NR} /^## License/{l=NR} END{print (h<v && v<l) ? "OK" : "FAIL"}' README.md
```

→ `OK`.

## Plan de pruebas

Este repo no tiene suite de tests tradicional. La verificación con contenido
real es que **`scripts/check.py`'s check5 pase con el número nuevo**, no
solo que los dos archivos contengan la misma cadena por casualidad — por eso
el Paso 0 confirma la concordancia *antes* del cambio y el Paso 2 la
confirma *después*, leyendo el mensaje completo (`'1.1.0'`, no solo
`all checks passed`).

Si `claude` CLI está disponible en tu entorno (instalado por el plan 011 si
ya aterrizó, o disponible localmente), confirmá que el manifiesto sigue
siendo válido tras el bump:

```sh
claude plugin validate . --strict 2>&1 | tail -3
```

No bloqueante si el comando no existe en tu entorno — `scripts/check.py` ya
cubre lo esencial (que `version` sea un campo no vacío y concuerde).

## Criterios de done

Comprobables por máquina. Deben cumplirse TODOS:

- [ ] `grep -c '"version": "1.1.0"' .claude-plugin/plugin.json` → `1`
- [ ] `grep -c '"version": "1.0.0"' .claude-plugin/plugin.json` → `0`
- [ ] `grep -c 'version: "1.1.0"' skills/improve/SKILL.md` → `1`
- [ ] `grep -c 'version: "1.0.0"' skills/improve/SKILL.md` → `0`
- [ ] `python3 scripts/check.py` → termina en `all checks passed`, y la salida contiene `check5: version agrees: '1.1.0'`
- [ ] `grep -c '^## Versioning' README.md` → `1`
- [ ] El comando `awk` del Paso 3 imprime `OK`
- [ ] `git diff --name-only main...HEAD` → exactamente estas tres rutas: `.claude-plugin/plugin.json`, `skills/improve/SKILL.md`, `README.md`
- [ ] Fila de estado actualizada en `plans/README.md`

> Nota sobre el chequeo de alcance: se usa `git diff --name-only main...HEAD`
> —no `git status`— porque este plan te pide commitear cada paso. Después de
> commitear, `git status` está limpio y no verificaría nada.

## Condiciones de STOP

Detente y reporta (no improvises) si:

- El Paso 0 muestra que `plugin.json` y `SKILL.md` **ya** discrepan antes de
  tu cambio (`check5` falla). Eso es un hallazgo distinto al de este plan —
  repórtalo, no lo arregles como si fuera parte de este bump.
- Los extractos de "Estado actual" no coinciden con el código vivo — el repo
  derivó desde `281903a`, y puede que la versión ya no sea `1.0.0` (alguien
  más pudo haber hecho un bump). Si es así, no apliques este plan sobre un
  número que ya cambió sin releer la situación primero.
- `scripts/check.py` deja de pasar después de tu bump por cualquier razón
  distinta a la versión (por ejemplo, tocaste sin querer otro campo de
  `plugin.json`). Revertí el cambio accidental antes de seguir.

## Notas de mantenimiento

Para quien tome este código después del cambio:

- **La política es prosa, no tooling.** No hay nada que fuerce a un futuro
  contribuyente a elegir el tamaño de bump correcto según la tabla del Paso
  3 — solo el chequeo de *concordancia* (`check5`) es mecánico. Si en algún
  momento se quiere hacer cumplir el *tamaño* del bump automáticamente
  (por ejemplo, detectar que `description` cambió y exigir al menos MINOR),
  eso es un plan nuevo, no una extensión de este.
- **Qué mirar en el review**: que el número nuevo sea `1.1.0` en ambos
  archivos a la vez — nunca uno solo. Es exactamente lo que `check5` ya
  vigila, pero vale la pena confirmarlo con los ojos además de con el script.
- **Deferred:** automatizar el bump como parte del flujo de `execute` o de
  `reconcile` (por ejemplo, que el índice de `plans/README.md` sugiera un
  tamaño de bump cuando detecta que un plan tocó `description` o una Hard
  Rule). No bloqueado por nada — es trabajo de diseño que no se hizo porque
  este plan se mantuvo deliberadamente al tamaño de "bump puntual +
  política escrita", no "sistema de versionado automatizado".
