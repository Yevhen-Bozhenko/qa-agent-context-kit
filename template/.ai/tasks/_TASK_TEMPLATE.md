# Task: <short imperative title>

<!--
How to use:
1. Copy this file to .ai/tasks/<yyyy-mm-dd>-<slug>.md
2. Fill the six sections. Delete hints. Leave a section empty only on purpose.
3. Tell the agent: "Read AGENTS.md, then execute .ai/tasks/<file>.md"

Only put task-specific information here. Anything that is true for every task
belongs in .ai/qa-context.md or .ai/framework-guide.md instead.
-->

## 1. Task

What to do, in one to three sentences. One outcome per task file.
<!-- e.g. "Add API tests for POST /bookings covering the validation rules below." -->

## 2. Context

What the agent needs to know that is *not* already in the persistent context.
<!--
- Links / pasted excerpts: ticket, acceptance criteria, API spec section, test case
- Relevant files to look at first (paths)
- Current state: what exists already, what is broken, error output, CI log excerpt
-->

## 3. Constraints

What the agent must not do or must stay within for *this* task.
<!--
- Scope: which files/folders may be changed; which may not
- No production code changes / no new dependencies / no changes to shared fixtures
- Time-boxing: "if you can't reproduce in N runs, stop and report"
-->

## 4. Examples

Concrete material to imitate or cover.
<!--
- "Model the new tests on tests/api/users.spec.ts"
- Input → expected-output pairs, sample payloads, the exact cases to cover
- A counter-example: "do NOT do it like tests/legacy/..."
-->

## 5. Expected output

What "done" looks like — the shape of the deliverable.
<!--
- Files to create/modify
- Number/list of test cases and their names
- Tags, annotations, report steps
- Format of the final report (default: the one in AGENTS.md)
-->

## 6. Validation

How the agent must prove the result, and what to report.
<!--
- Commands to run and the expected outcome
- Negative check: "temporarily break X / reason about it — the test must fail with <message>"
- Stability: "run N times"
- What to do if validation fails: fix, or stop and report — never weaken the test
-->
