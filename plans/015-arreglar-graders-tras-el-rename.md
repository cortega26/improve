# Plan 015: Fix the eval graders and license wording after the 014 rename

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
> **Drift check (run first)**: `git diff --stat 5f9f6d5..HEAD -- evals/fires-on-repo-audit-request/graders/skill-invoked.md evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md evals/README.md README.md LICENSE.md`
> If any in-scope file changed since this plan was written, compare the
> "Current state" excerpts below against the live files before proceeding; on
> a mismatch, treat it as a STOP condition.

## Status

- **Priority**: P3
- **Effort**: S
- **Risk**: LOW
- **Depends on**: plans/014-renombrar-y-posicionar-el-fork.md (done — this plan fixes fallout from it)
- **Category**: tech-debt
- **Planned at**: commit `5f9f6d5`, 2026-08-18

## Why this matters

Plan 014 renamed this fork's plugin and skill identity from `improve` to
`improve-cortega26`. Two files it correctly left out of scope now contain a
real, if currently dormant, defect: the eval suite's grader regexes
(`plans/011-suite-de-evals.md`'s deliverable) anchor on the literal string
`improve"`, which no longer matches an invocation of the renamed skill. The
practical impact today is zero — `claude plugin eval` is still gated behind
early access in this environment, so nothing runs these graders — but the
defect is real, cheap to fix, and worth fixing now while the cause is fresh
rather than as a mystery failure whenever early access opens up. This plan
also aligns a small wording mismatch between `LICENSE.md` and `README.md`'s
fork-copyright lines, found during the same review.

## Current state

- `evals/fires-on-repo-audit-request/graders/skill-invoked.md` (full file):
  ```yaml
  ---
  type: tool_used
  tool: Skill
  input_match: '"skill"\s*:\s*"(?:[\w-]+:)?improve"'
  min: 1
  ---
  A full-repo audit request naming multiple of the skill's stated categories
  (tech debt, security, direction) should trigger the `improve` skill.
  ```
- `evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md` (full file):
  ```yaml
  ---
  type: tool_used
  tool: Skill
  input_match: '"skill"\s*:\s*"(?:[\w-]+:)?improve"'
  min: 0
  max: 0
  ---
  The `improve` skill must not fire for a bare request to review a diff or
  uncommitted changes — that's explicitly out of scope per SKILL.md's "When
  not to use this" section, except through the `branch` invocation variant,
  which was not requested here.
  ```
- `evals/README.md:1`: `` # Eval suite for the `improve` skill ``
- `README.md:157`: `MIT © 2026 Carlos Ortega (fork-specific changes)`
- `LICENSE.md:5`: `Portions Copyright (c) 2026 Carlos Ortega (cortega26 fork)`
- **Why the regex is broken, verified, not assumed**: the `Skill` tool's own
  parameter documentation states plugin-sourced skills are identified as
  `plugin:skill` in tool-call input. Post-014, this fork's plugin name and
  skill name are both `improve-cortega26`, so a qualified invocation logs as
  `"improve-cortega26:improve-cortega26"`; an unqualified one (if the host
  ever logs bare skill names) would log as `"improve-cortega26"`. Neither
  ends in the literal substring `improve"` that the current regex requires —
  both end in `improve-cortega26"`. The existing `(?:[\w-]+:)?` prefix group
  already handles "optionally qualified" generically and needs no change;
  only the literal suffix needs to grow from `improve` to `improve-cortega26`.

## Commands you will need

| Purpose | Command | Provenance | Expected on success |
|---|---|---|---|
| Regex sanity check (no eval run needed) | `python3 -c "import re; assert re.search(r'\"skill\"\s*:\s*\"(?:[\w-]+:)?improve-cortega26\"', '{\"skill\": \"improve-cortega26:improve-cortega26\"}'); assert re.search(r'\"skill\"\s*:\s*\"(?:[\w-]+:)?improve-cortega26\"', '{\"skill\": \"improve-cortega26\"}'); print('ok')"` | executed | `ok` |
| Structural checks (unaffected by this plan, confirms nothing broke) | `python3 scripts/check.py` | executed | `all checks passed`, exit 0 |

Both run against the unmodified checkout at `5f9f6d5` before writing this
plan; the regex check confirms the *new* pattern (not yet written to disk)
matches both invocation shapes described above.

## Scope

**In scope** (the only files you should modify):
- `evals/fires-on-repo-audit-request/graders/skill-invoked.md`
- `evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md`
- `evals/README.md` (only the `# Eval suite for the improve skill` title on line 1)
- `README.md` (only line 157, the fork-copyright line)

**Out of scope** (do NOT touch, even though they look related):
- `skills/improve/references/plan-template.md:202` and
  `skills/improve/references/closing-the-loop.md:124` — generic prose
  mentions of "the improve skill" found during 014's review. Cosmetic only,
  not load-bearing for any check or eval, and not worth a plan of their own.
- `plans/011-suite-de-evals.md` — its recorded target text for the grader
  files will become stale after this plan lands. That is expected and
  handled in Maintenance notes below, not fixed by editing the historical
  plan file.
- `LICENSE.md` — already correct; only `README.md`'s wording changes to
  match it, not the other way around (see Step 3's rationale).
- `claude plugin eval` itself — still gated behind early access in this
  environment; this plan cannot verify the grader against a real run, only
  that the regex is structurally correct (see Commands table).

## Git workflow

- Branch: `advisor/015-fix-graders-post-rename`
- Commit per step (three steps below → three commits), conventional-commit
  style matching this repo's history (`git log --oneline`, e.g. `fix:`, `docs:`).
- Do NOT push or open a PR unless the operator instructed it.

## Steps

### Step 0: Establish a green baseline

Run both commands from the table above on the unmodified checkout.

- If both pass as expected: proceed to Step 1.
- If either fails: the repo drifted since this plan was written. **STOP and
  report** the exact output — do not proceed with a red baseline.

**Verify**: regex sanity check → `ok`. `python3 scripts/check.py` → `all
checks passed`, exit 0.

### Step 1: Fix both grader `input_match` regexes

In both grader files, change:
```
input_match: '"skill"\s*:\s*"(?:[\w-]+:)?improve"'
```
to:
```
input_match: '"skill"\s*:\s*"(?:[\w-]+:)?improve-cortega26"'
```
Do not touch `type`, `tool`, `min`, `max`, or the prose body of either file.

**Verify**: `grep -c 'improve-cortega26"' evals/fires-on-repo-audit-request/graders/skill-invoked.md evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md` → `1` for each file. `grep -rn '"(?:\[\\w-\]+:)?improve"' evals/` → no matches (the old literal is gone from both).

### Step 2: Update `evals/README.md`'s title

Change line 1 from `` # Eval suite for the `improve` skill `` to
`` # Eval suite for the `improve-cortega26` skill ``. Do not touch anything
else in the file (the "Status: unverified" section and everything below it
is unrelated to the rename).

**Verify**: `grep -c 'improve-cortega26' evals/README.md` → at least `1`.

### Step 3: Align `README.md`'s fork-copyright wording with `LICENSE.md`

`LICENSE.md:5` already reads `Portions Copyright (c) 2026 Carlos Ortega
(cortega26 fork)`. `README.md:157` says the same thing with different
words: `MIT © 2026 Carlos Ortega (fork-specific changes)`. Two phrasings of
one legal claim in two files is exactly the kind of drift `scripts/check.py`
exists to catch elsewhere and doesn't catch here. Change `README.md:157` to
match `LICENSE.md` exactly:
```
MIT © shadcn
Portions © 2026 Carlos Ortega (cortega26 fork)
```
(keep the `MIT © shadcn` line above it unchanged — only the second line's
wording changes, from "MIT © 2026 Carlos Ortega (fork-specific changes)" to
"Portions © 2026 Carlos Ortega (cortega26 fork)").

**Verify**: `grep -c "Portions.*Carlos Ortega (cortega26 fork)" README.md` → `1`.

## Test plan

There is no executable test suite in this repo — verification is the
structural checker and the regex sanity check, both run after every step
above and again in Done criteria. No new test files are created.

## Done criteria

Machine-checkable. ALL must hold:

- [ ] `python3 scripts/check.py` exits 0, prints `all checks passed`
- [ ] `grep -c 'improve-cortega26"' evals/fires-on-repo-audit-request/graders/skill-invoked.md` → `1`
- [ ] `grep -c 'improve-cortega26"' evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md` → `1`
- [ ] `grep -c 'improve-cortega26' evals/README.md` → at least `1`
- [ ] `grep -c "Portions.*Carlos Ortega (cortega26 fork)" README.md` → `1`
- [ ] `python3 -c "import re; assert re.search(r'\"skill\"\s*:\s*\"(?:[\w-]+:)?improve-cortega26\"', '{\"skill\": \"improve-cortega26:improve-cortega26\"}'); print('ok')"` → `ok`
- [ ] `git diff --name-only main...HEAD` lists exactly: `evals/fires-on-repo-audit-request/graders/skill-invoked.md`, `evals/does-not-fire-on-code-review-request/graders/no-skill-invocation.md`, `evals/README.md`, `README.md` — nothing else
- [ ] `plans/README.md` status row for 015 updated

## STOP conditions

Stop and report back (do not improvise) if:

- The code at the locations in "Current state" doesn't match the excerpts
  (the codebase has drifted since this plan was written).
- A step's verification fails twice after a reasonable fix attempt.
- The fix appears to require touching an out-of-scope file, in particular
  `plans/011-suite-de-evals.md` — its now-stale target text is handled by a
  maintenance note, not an edit to the plan file itself.
- A tool or sandbox blocks a command in this plan — report the block
  verbatim and stop. Do not attempt to quote-split, script around, or
  otherwise evade the block.

## Maintenance notes

- `plans/011-suite-de-evals.md`'s "Current state"/target-text excerpts for
  both grader files are now stale (they show the pre-015 regex verbatim).
  This is expected — 011 is DONE and is a historical record, same as 007's
  frozen example. Do not edit 011 to keep it in sync; if `reconcile` flags
  the mismatch later, this note is the explanation.
- **Deferred**: the generic "the improve skill" prose mentions in
  `skills/improve/references/plan-template.md:202` and
  `closing-the-loop.md:124` (found during 014's review) are still
  unaddressed. Nothing blocks fixing them; they were left out of this
  plan's scope only because they are cosmetic, not because they need
  anything else to land first.
- **Deferred**: this plan cannot verify the grader against a real
  `claude plugin eval` run — still gated behind early access in this
  environment (see `evals/README.md`'s own "Status" section). Whoever gets
  access first should run it against both eval cases and correct the
  grader schema for real if it turns out to differ from what plan 011 and
  this plan assumed.
