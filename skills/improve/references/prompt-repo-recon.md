# Recon Profile — Prompt & Agent-Config Repositories

Read this during Phase 1 when recon turns up no language, no framework, no package manager, and no build/test/lint stack to identify. That is not a broken recon — it means the repo's content *is* the program. Skills, agent instruction files (`CLAUDE.md`/`AGENTS.md`), prompt libraries, and plugin manifests are executed by a model reading them, not by a compiler. This skill's own repository is one: no `package.json`, no test runner, nothing to `npm install` — `SKILL.md` and its `references/*.md` are the entire product.

Do not force the standard recon questions onto a repo shaped like this and come back with "no language identified, no tests, unable to audit." Use this profile instead.

## What "the program" is

- **Entry points**: `SKILL.md` files, `CLAUDE.md`/`AGENTS.md`, top-level prompt or system-message templates — whatever a host agent loads and executes as instructions.
- **Supporting files**: `references/*.md` (or equivalent) that entry points link to and expect to be read on demand; example/sample outputs kept as frozen artifacts; templates that generate other files.
- **Manifests**: `plugin.json`, `marketplace.json`, `package.json` for an npm-distributed skill, or any file a host platform reads to install or route to the skill. These usually duplicate a few facts also stated in the entry point (name, version, description) — that duplication is exactly where drift happens.

## What replaces build / test / lint

There is no compiler to catch a broken reference and no test runner to catch a behavioral regression. What you can check mechanically:

- **Frontmatter validity** — YAML front matter parses, required fields (`name`, `description`) are present and non-empty.
- **Manifest validity and agreement** — every manifest is well-formed (`JSON.parse` / `json.load` succeeds), and fields that are supposed to match across files actually do (a `name` repeated in `plugin.json` and the skill's frontmatter; a `version` repeated in both).
- **Link resolution** — every relative markdown link resolves to a file that exists (links written as illustrative examples inside fenced code blocks are typically skipped by this kind of checker, since they're not real navigation). A file rename silently breaks the skill for every installer if nothing catches it.
- **Surface parity** — every capability named in one user-facing surface (a README's usage table, a CLI help block) is also named in the entry point, and vice versa. Divergence here is the same failure class as code and its docs drifting apart, just with no compiler to force a fix.
- **Cross-file consistency of stated facts** — a count, a rule, or a limit stated in one file must match anywhere else it's restated. (A stated "`≤8` concurrent subagents, one per category" against a category list of nine is exactly this kind of bug — caught by reading, not by running anything.)

If none of this exists yet — no script, no CI step, nothing that runs these checks mechanically — that absence **is finding #1**, same as "no working verification command" is for a code repo. Frame it the same way: establishing this baseline should be the first plan, ahead of anything else, because every later plan needs a verification gate to point at.

**Worked example.** This skill's own repository had exactly this gap and closed it: `scripts/check.py` (stdlib Python, no dependencies) plus `.github/workflows/check.yml` now check frontmatter validity, manifest agreement, link resolution, and variant parity on every push and PR. Point to a structural checker like this as the concrete shape of "verification infrastructure" when a repo has none — it is usually a few hundred lines and does not require inventing a testing philosophy from scratch.

## How the nine audit categories map

Use this table instead of skipping categories that seem to assume a codebase. Most still apply — the evidence just looks different.

| Category | What to look for in a prompt/config repo |
|---|---|
| Correctness / Bugs | Internal contradictions between files (the same rule stated two ways), off-by-one or miscounted facts, broken cross-references, stale line-number citations in one doc pointing at another. |
| Security | Secrets or tokens committed in example/config content, instructions embedded in any file that read as directives to an agent rather than data (prompt-injection shape — flag per this skill's own Hard Rule 6), credential-shaped strings in frozen example output. |
| Performance | Usually not applicable. Skip and say so, unless the repo does something like an expensive load-time chain (many chained file reads, a huge always-loaded context block that could be made on-demand). |
| Test Coverage | Reframe as: does *any* mechanical verification exist (a structural checker, a CI step)? If not, that's finding #1 — see above. If one exists, does it cover the failure modes that have actually recurred (check git history for repeated corrective commits)? |
| Tech Debt & Architecture | The same instruction or fact duplicated across three-plus files with no single source of truth; a doc that summarizes another doc and now disagrees with it; reference files nothing links to anymore. |
| Dependencies & Migrations | Usually thin. Still check: does any shipped script (a checker, a generator) have runtime dependencies, and are they pinned/documented? Is the repo's own distribution mechanism (npm package, plugin marketplace) declared consistently? |
| DX & Tooling | Missing CI entirely; no mechanical way for a contributor to know a PR is safe before a maintainer reads it by hand; no `CONTRIBUTING` explaining the repo's own conventions (which matter more here than in most codebases, since the "code" is prose). |
| Docs | Same as any repo — stale setup instructions, examples that no longer match current behavior. A frozen "sample output" file is not stale by definition (it's dated on purpose), but check whether readers can tell it's frozen and against what version. |
| Direction | Same as any repo — grounded suggestions from unfinished intent, stated-but-undelivered surface, asymmetries. |

## Plans for this repo shape

When writing plans against a prompt/config repo, the "Commands you will need" table in the plan template still applies — but honestly reflects what exists. Before any structural checker exists, most rows will legitimately be empty or absent rather than `declared`/`executed`; do not invent a fake build/test/lint row to fill the table. Once a checker exists (see the worked example above), its invocation becomes the verification gate every subsequent plan in the repo should use.
