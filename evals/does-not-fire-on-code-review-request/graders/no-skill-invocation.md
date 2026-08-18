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
