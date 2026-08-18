# Plan 014: Rename the fork's invocation namespace and position it as a fork

> **Executor instructions**: Follow this plan step by step. Run every
> verification command and confirm the expected result before moving to the
> next step. If anything in the "STOP conditions" section occurs, stop and
> report — do not improvise. When done, update the status row for this plan
> in `plans/README.md` — unless a reviewer dispatched you and told you they
> maintain the index.
>
> **Never bypass, quote-split, or route around a tool/sandbox security
> block.** If a command is blocked, report the block verbatim and STOP — do
> not construct a workaround to force it through. This has happened before in
> this project's history (see `plans/README.md`, "Hallazgo de seguridad —
> ejecución de plan 011") and is treated as a serious trust violation
> independent of whether the underlying command was harmless.
>
> **Drift check (run first)**: `git diff --stat 366bb03..HEAD -- .claude-plugin/plugin.json .claude-plugin/marketplace.json skills/improve/SKILL.md README.md LICENSE.md scripts/check.py`
> If any in-scope file changed since this plan was written, compare the
> "Current state" excerpts below against the live files before proceeding; on
> a mismatch, treat it as a STOP condition.

## Status

- **Priority**: P2
- **Effort**: M
- **Risk**: LOW
- **Depends on**: plans/009-ci-estructural.md (done — this plan patches a bug in the checker it introduced)
- **Category**: tech-debt
- **Planned at**: commit `366bb03`, 2026-08-18

## Why this matters

This repo's plugin (`plugin.json:name`) and skill (`SKILL.md:name`) are both
literally `improve` — identical to the unmaintained upstream
(`shadcn/improve`) and identical to whatever copy a user already has
installed globally. This has already caused a real, observed failure in this
project's own session history: a June-2026 snapshot at
`~/.claude/skills/improve` won invocation-resolution over this repo's copy
when both were present, because nothing distinguishes them by name. Beyond
the collision, `plugin.json`'s `homepage`/`repository` still point at
`shadcn/improve` even though this fork now lives at a different remote, and
the license carries only the original author's copyright with no trace that
a fork exists. This plan fixes the namespace collision and brings the
plugin/marketplace identity and licensing into line with "this is
`cortega26`'s fork," while preserving the original author's attribution
exactly as the MIT license requires — added to, never replaced.

**Confirmed decisions (already made, not open questions for the executor):**
new invocation name is **`improve-cortega26`**, used identically for
`plugin.json:name`, `marketplace.json`'s top-level `name` and its
`plugins[0].name`, and `SKILL.md:name`. `homepage`/`repository` in
`plugin.json` move to `https://github.com/cortega26/improve`. The original
author (`shadcn`) stays as `plugin.json:author` and in `LICENSE.md`'s
existing copyright line — nothing about original authorship is removed.

## Current state

- `.claude-plugin/plugin.json` (full file, 12 lines):
  ```json
  {
    "name": "improve",
    "description": "Point it at a codebase and it figures out what's worth doing — bugs, perf, tech debt, what to build next — then writes plans any agent can execute. It can hand the work to a cheaper model and review the result. It never edits your code.",
    "version": "1.1.0",
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
- `.claude-plugin/marketplace.json` (full file, 15 lines):
  ```json
  {
    "name": "improve",
    "owner": {
      "name": "shadcn"
    },
    "metadata": {
      "description": "Claude marketplace catalog for the improve plugin."
    },
    "plugins": [
      {
        "name": "improve",
        "source": "./"
      }
    ]
  }
  ```
- `skills/improve/SKILL.md:1-8` (frontmatter block):
  ```yaml
  ---
  name: improve
  description: Survey any codebase as a senior advisor and produce prioritized, self-contained implementation plans for OTHER models/agents to execute. Strictly read-only on source code — never implements, fixes, or refactors anything itself. Use when asked to audit a codebase, find improvement opportunities (bugs, security, performance, test coverage, tech debt, migrations, DX), suggest features or where to take the project next (roadmap, product direction), or generate handoff plans for another agent to implement. Do NOT use for reviewing a diff, a PR, or uncommitted changes (that is a code-review skill's job), for implementing or refactoring anything, or for planning a feature the user has already scoped — this skill audits a repository at rest and produces specs for others to execute.
  license: MIT
  metadata:
    author: shadcn
    version: "1.1.0"
  ---
  ```
  Only `name:` (line 2) changes. `description`, `license`, `metadata.author`,
  `metadata.version` are unrelated to this plan and must not change — an
  unrelated change to `description` is at least a MINOR version bump per this
  repo's own versioning policy (`README.md`, "## Versioning"), which is out
  of scope here.
- `scripts/check.py:71-82` — **this is a real blocker, not optional
  cleanup**. Check 1 hardcodes the literal string `"improve"` as the only
  legal `SKILL.md` name:
  ```python
          # name value must be "improve"
          name_match = None
          for l in fm_lines:
              m = re.match(r"^name:\s+(\S.*)", l)
              if m:
                  name_match = m.group(1).strip()
                  break
          if name_match != "improve":
              fail(f"check1: SKILL.md frontmatter name is {name_match!r}, expected 'improve'")
          else:
              if has_name and has_desc:
                  ok("check1: SKILL.md frontmatter valid (name='improve', description present)")
  ```
  This script was cherry-picked verbatim from the upstream repo (plan 009) —
  upstream never needed this to be generic because it never renames itself.
  Left as-is, renaming the skill makes CI fail permanently. Check 2
  (`scripts/check.py:131-136`) already verifies `plugin.json` name and
  `SKILL.md` name agree with each other — that cross-file check is the
  correct, name-agnostic replacement for check 1's hardcoded literal.
- `README.md` — every `/improve` occurrence that is the literal invocation
  token (not prose) must move to `/improve-cortega26`. Exact lines as of
  `366bb03`:
  - Line 1: `# improve`
  - Line 8: `you          →  /improve                    (expensive model, advises)`
  - Line 16: `npx skills add shadcn/improve`
  - Line 22: `/plugin marketplace add shadcn/improve`
  - Line 23: `/plugin install improve@improve`
  - Lines 31-41 (11 lines, the full `## Usage` block): each starts with
    `/improve` followed by a variant keyword or nothing, e.g.
    `/improve quick                  cheap pass: hotspots, top findings only`
  - Line 46: `` `improve` audits a repository at rest and writes specs. ``
  - Line 49: `` scoped, a general planning skill is cheaper. `improve branch` is the one ``
  - Line 57: `` 1. Open your agent in the repo and run `/improve` (or `/improve quick` to keep it cheap). ``
  - Line 60: `` 4. Hand a plan to any agent ("implement plans/001-*.md"), or let the skill run it: `/improve execute 001`. ... ``
  - Line 61: `` 5. Next session, run `/improve reconcile` to clean up the backlog: ... ``
  - Line 63: `` Before a PR, `/improve branch` does the same thing scoped to just what your branch changes. ``
  - Line 147: `` the routing trigger a host agent reads to decide whether to invoke `/improve` ``
  - Line 154 (`## License`): `MIT © shadcn`
- `LICENSE.md` (full file, 20 lines) — the only relevant line is `Copyright
  (c) 2026 shadcn` (line 3). It must remain, unmodified, above the added
  fork line.

## Commands you will need

| Purpose | Command | Provenance | Expected on success |
|---|---|---|---|
| Structural checks | `python3 scripts/check.py` | executed | `all checks passed`, exit 0 |
| Plugin/marketplace manifest validation | `claude plugin validate . --strict` | executed | `✔ Validation passed`, exit 0 |

Both were run against the unmodified checkout at `366bb03` immediately before
writing this plan and passed cleanly (see Step 0).

## Scope

**In scope** (the only files you should modify):
- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`
- `skills/improve/SKILL.md` (frontmatter `name:` line only)
- `README.md`
- `LICENSE.md`
- `scripts/check.py` (only the hardcoded-literal fix in check 1, lines 71-82)

**Out of scope** (do NOT touch, even though they look related):
- `examples/001-extract-shadow-config-resolution.md` — its header explicitly
  says "A real plan produced by `/improve` against shadcn/ui at commit
  `1994caba0`" and "run `/improve` on your own repo instead." This is a
  historical record (plan 007 deliberately froze it rather than keep it in
  sync with template/identity changes going forward) — the first mention is
  a fact about the past and must stay accurate to what actually happened;
  the second is now slightly stale advice, which is acceptable staleness in
  a frozen example, not a bug to fix here. Leave the whole file untouched.
- `README.md:67` — `` A run against [shadcn/ui](https://github.com/shadcn-ui/ui) came back with findings like: `` — `shadcn/ui` here names the audited *example* codebase used to illustrate output, not this project's own identity. The string match is coincidental; do not rename it.
- `plugin.json:author` and `SKILL.md:metadata.author` — stay `shadcn`. This
  is attribution for who wrote the original content, independent of who
  hosts/maintains this fork's repository and marketplace listing.
- Any file under `plans/` other than this plan's own status row in
  `plans/README.md`.
- `scripts/check.py` checks 2 through 5 and anything outside the exact lines
  identified in Step 1 below.

## Git workflow

- Branch: `advisor/014-rename-fork`
- Commit per step (six steps below → six commits), conventional-commit style
  matching this repo's history (`git log --oneline`, e.g. `fix:`, `docs:`,
  `chore:`).
- Do NOT push or open a PR unless the operator instructed it.

## Steps

### Step 0: Establish a green baseline

Run both commands from the table above on the unmodified checkout.

- If both pass as expected: proceed to Step 1.
- If either fails: the repo drifted since this plan was written (both were
  `executed`-verified green at `366bb03`). **STOP and report** the exact
  output — do not proceed with a red baseline.

**Verify**: `python3 scripts/check.py` → `all checks passed`, exit 0.
`claude plugin validate . --strict` → `✔ Validation passed`, exit 0.

### Step 1: Generalize `scripts/check.py` check 1 (unblocks every later step)

Replace the hardcoded-to-`"improve"` name assertion at
`scripts/check.py:71-82` with a check that only requires the name be
non-empty (already guaranteed by `has_name` above it) — cross-file agreement
with `plugin.json` is already handled by check 2, so check 1 should not
duplicate it with a literal string. Target shape:

```python
        # name value: non-empty is already required by has_name above.
        # Cross-file agreement with plugin.json's name is check 2's job —
        # this repo may be forked/renamed, so no literal value is enforced here.
        name_match = None
        for l in fm_lines:
            m = re.match(r"^name:\s+(\S.*)", l)
            if m:
                name_match = m.group(1).strip()
                break
        if has_name and has_desc and name_match:
            ok(f"check1: SKILL.md frontmatter valid (name={name_match!r}, description present)")
```

Do this step **before** renaming anything else, so the checker never fails
transiently mid-sequence.

**Verify**: `python3 scripts/check.py` → still `all checks passed` (name is
still literally `improve` at this point, so check 1's new, looser condition
must also pass on the unmodified name — confirming you didn't break the
check, only loosened the one over-constrained assertion).

### Step 2: Rename in `.claude-plugin/plugin.json`

Change exactly three values, leave everything else (including `author`)
identical:
- `"name": "improve"` → `"name": "improve-cortega26"`
- `"homepage": "https://github.com/shadcn/improve"` → `"homepage": "https://github.com/cortega26/improve"`
- `"repository": "https://github.com/shadcn/improve"` → `"repository": "https://github.com/cortega26/improve"`

**Verify**: `python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); assert d['name']=='improve-cortega26'; assert d['homepage']=='https://github.com/cortega26/improve'; assert d['author']['name']=='shadcn'; print('ok')"` → `ok`

### Step 3: Rename in `.claude-plugin/marketplace.json`

Change exactly three values:
- Top-level `"name": "improve"` → `"name": "improve-cortega26"`
- `"owner": {"name": "shadcn"}` → `"owner": {"name": "cortega26"}` (this is
  who maintains the marketplace *listing*, distinct from `plugin.json`'s
  `author`, which stays `shadcn` as the original content's author)
- `"plugins"[0]"name": "improve"` → `"name": "improve-cortega26"` (must match
  `plugin.json`'s new name exactly, or `claude plugin validate` will flag a
  mismatch)

**Verify**: `claude plugin validate . --strict` → `✔ Validation passed`, exit 0.

### Step 4: Rename in `skills/improve/SKILL.md`

Change only line 2: `name: improve` → `name: improve-cortega26`. Do not
touch `description`, `license`, or the `metadata` block.

**Verify**: `python3 scripts/check.py` → `all checks passed`, and specifically
`check2: plugin.json valid, name='improve-cortega26' matches SKILL.md` (not
the old `check1` message — that assertion no longer names a literal value
after Step 1).

### Step 5: Update `README.md`

Two kinds of edits:

1. **Invocation rename** — replace `/improve` with `/improve-cortega26` at
   every line listed in "Current state" above (1, 8, 16, 22, 23, 31-41, 46,
   49, 57, 60, 61, 63, 147). Line 23 becomes
   `/plugin install improve-cortega26@improve-cortega26`; line 16 becomes
   `npx skills add cortega26/improve`; line 22 becomes
   `/plugin marketplace add cortega26/improve`. Do not touch line 67
   (`shadcn/ui` example target — see Scope).

2. **Fork disclosure + license line** — immediately under the `# improve-cortega26`
   H1 (before the "An agent skill that..." paragraph), add one line:
   ```
   > Fork of [shadcn/improve](https://github.com/shadcn/improve).
   ```
   And in the `## License` section (currently just `MIT © shadcn` at line
   154), add a second line beneath it — do not replace the first:
   ```
   MIT © shadcn
   MIT © 2026 Carlos Ortega (fork-specific changes)
   ```

**Verify**: `python3 scripts/check.py` → `all checks passed` including
`check4: all 9 variants present in both README.md and SKILL.md` (the
variant keywords like `quick`/`execute`/`reconcile` are untouched by this
rename — only the `/improve` prefix changes — so parity must still hold).
`grep -c "^/improve " README.md` → `0` (no line should still start with the
old bare invocation prefix followed by a space; `/improve-cortega26` lines
do not match this pattern).

### Step 6: Update `LICENSE.md`

Add one line directly after the existing copyright line (line 3) — do not
remove or edit the original:

```
Copyright (c) 2026 shadcn

Portions Copyright (c) 2026 Carlos Ortega (cortega26 fork)
```

**Verify**: `grep -c "Copyright (c) 2026 shadcn" LICENSE.md` → `1` (original
line still present, unmodified). `grep -c "Carlos Ortega" LICENSE.md` → `1`
(new line added).

## Test plan

There is no executable test suite in this repo (it is prompt/config
content) — verification is the structural checker and the plugin validator,
both run after every step above and again in Done criteria. No new test
files are created by this plan.

## Done criteria

Machine-checkable. ALL must hold:

- [ ] `python3 scripts/check.py` exits 0, prints `all checks passed`
- [ ] `claude plugin validate . --strict` exits 0, prints `✔ Validation passed`
- [ ] `python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); assert d['name']=='improve-cortega26'; assert d['author']['name']=='shadcn'; assert d['repository']=='https://github.com/cortega26/improve'; print('ok')"` → `ok`
- [ ] `grep -c '^name: improve-cortega26$' skills/improve/SKILL.md` → `1`
- [ ] `grep -c "Copyright (c) 2026 shadcn" LICENSE.md` → `1` (original preserved)
- [ ] `grep -c "improve-cortega26" README.md` → at least `14` (H1 + 13 renamed invocation lines, install commands counted separately by their own literal strings)
- [ ] `git diff --name-only main...HEAD` lists exactly: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `skills/improve/SKILL.md`, `README.md`, `LICENSE.md`, `scripts/check.py` — nothing else (three dots — compares against the merge base, so it still holds if other work landed on `main` meanwhile). `git status` is not a scope check here: this plan tells you to commit each step, and committed work leaves it clean.
- [ ] `plans/README.md` status row for 014 updated

## STOP conditions

Stop and report back (do not improvise) if:

- The code at the locations in "Current state" doesn't match the excerpts
  (the codebase has drifted since this plan was written).
- A step's verification fails twice after a reasonable fix attempt.
- The fix appears to require touching an out-of-scope file — in particular,
  do NOT touch `examples/001-extract-shadow-config-resolution.md` even if it
  looks inconsistent after the rename; that inconsistency is expected and
  accepted (see Scope).
- You discover `scripts/check.py` has other hardcoded assumptions about the
  literal name `"improve"` beyond check 1 (e.g. in checks 3-5) — the plan's
  own reading of the file at `366bb03` found none, but if the file drifted,
  treat new ones as drift, not something to silently patch beyond what Step
  1 specifies.
- A tool or sandbox blocks a command in this plan — report the block
  verbatim and stop. Do not attempt to quote-split, script around, or
  otherwise evade the block (see the security-incident note linked at the
  top of this plan).

## Maintenance notes

- After this lands, anyone with the old global `~/.claude/skills/improve`
  copy still installed will have two independent skills: their own
  (unaffected, still named `improve`) and this fork (`improve-cortega26`).
  That is the intended fix — they no longer collide.
- `examples/001-extract-shadow-config-resolution.md` will read as slightly
  inconsistent after this lands (it still says `/improve`, matching the name
  at the time it was generated). This is intentional — see Scope. **Deferred:**
  if the frozen example is ever regenerated or retired for other reasons,
  update its invocation references to match at that point — not as a
  standalone follow-up, since the whole point of freezing it was to avoid
  busywork keeping it in sync with every naming/template change.
- If Fase C's broader public-fork positioning (option C from the original
  fork-strategy decision) moves forward with a distinct public brand name
  later, `improve-cortega26` may need a second rename. **Deferred:** no
  action now — this plan intentionally chose the mechanical,
  collision-avoiding name over inventing a public brand, since that
  decision was explicitly out of scope for this session.
- A reviewer should scrutinize: that `plugin.json:author` and
  `SKILL.md:metadata.author` were left as `shadcn` (attribution must survive
  the rename), and that `LICENSE.md`'s original copyright line is byte-for-byte
  unchanged (only a new line was added below it).
