# Eval suite for the `improve` skill

Behavioral tests for the skill's routing (does it trigger on the right
requests and stay silent on the wrong ones) and, eventually, its adherence
to its own Hard Rules. Run with `claude plugin eval` — see below.

## Status: unverified, blocked on early-access enablement

As of `claude --version` 2.1.221, `claude plugin eval` (and its `init`
subcommand, which would generate the canonical case template) are gated
behind early access in this environment — both fail with `` `plugin eval` is
currently in early access `` and exit 1. The two cases in this directory were
written against the command's `--help` output and general documentation, not
against a successful run or a generated template. **Treat their exact YAML
schema as unverified** until someone with access runs `claude plugin eval
init --bare sanity-check` and diffs its output against the files here.

## Running the suite (once enabled)

```sh
claude plugin eval evals/does-not-fire-on-code-review-request
claude plugin eval evals/fires-on-repo-audit-request
```

Or all cases at once, from the plugin root:

```sh
claude plugin eval .
```

## What's covered

| Case | What it checks | Grounded in |
|---|---|---|
| `does-not-fire-on-code-review-request` | The skill does NOT trigger on a bare diff-review request | `SKILL.md`'s "When not to use this" section (plan 003) |
| `fires-on-repo-audit-request` | The skill DOES trigger on a full-repo audit request | `SKILL.md`'s frontmatter `description` |

## Known limitation

The eval sandbox loads only the plugin under test — so the negative case
proves the skill doesn't self-fire on an out-of-scope prompt in isolation,
not that it would lose a routing contest against a real competing
code-review skill loaded alongside it. That stronger test — multiple plugins
loaded together — is possible per the documented `plugins:` frontmatter
field but wasn't attempted here, since it couldn't be verified either.

## What's not covered yet

Hard Rule 1 (never edits source code) and Hard Rule 4 (never reproduces
secret values) would make strong eval cases — asserting the skill never
calls `Write`/`Edit` outside `plans/`, or never echoes a value matching a
credential pattern — but both need `--allow-tools Write Edit` and a scaffold
repo with a fixture secret, which raises the complexity past what could be
written without being able to test it. Left for a follow-up once the basic
two-case mechanism is confirmed working.
