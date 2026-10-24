# QA Agent Context Kit

A copy-paste template for getting **reliable, repeatable** results from AI coding agents
(Claude Code, Cursor, GitHub Copilot, Codex and others) on QA and test-automation work.

Companion repository for the webinar **Getting Better Results from AI Coding Agents in QA**.

---

## Why

Coding agents are good at writing *plausible* tests. Without context they also:

- write generic code next to your framework instead of inside it,
- invent helpers, selectors and validation rules,
- "fix" flaky tests with sleeps and retries,
- assert on status codes and visibility instead of the actual requirement,
- forget every correction the moment the chat ends.

Most of that is not a model problem — it's a missing-context problem. This kit gives the
agent the context once, in files that live in your repo.

## The pattern

```
AGENTS.md                    entry point: what to read, working rules, how to report
.ai/qa-context.md            persistent project context (stack, commands, conventions, never-do)
.ai/framework-guide.md       how to extend YOUR framework (reference tests, layers, reuse order)
.ai/tasks/<task>.md          one task: Task → Context → Constraints → Examples → Expected Output → Validation
```

Rule of thumb: **if you'd have to type it again for the next task, it belongs in a persistent file.**

Full explanation: [docs/pattern.md](docs/pattern.md)

## Quick start (≈15 minutes)

1. Copy everything inside [`template/`](template/) into the root of your test repository
   (including the hidden `.ai/` and `.github/` folders):

   ```bash
   git clone https://github.com/Yevhen-Bozhenko/qa-agent-context-kit.git /tmp/qa-kit
   cd /path/to/your-test-repo

   # 1. See which files you already have — these will NOT be copied
   for f in AGENTS.md CLAUDE.md .github/copilot-instructions.md .ai/qa-context.md .ai/framework-guide.md; do
     [ -e "$f" ] && echo "already exists, merge by hand: $f"
   done

   # 2. Copy without overwriting anything
   cp -rn /tmp/qa-kit/template/. .
   ```

   > ⚠️ `cp -n` silently skips files that already exist. If step 1 listed any file, your project
   > already has agent instructions: open both versions and merge the template's sections
   > into yours by hand, rather than replacing them.

2. Fill the `TODO`s in `.ai/qa-context.md` — start with *How to run*, *Conventions* and *Never do*.
3. In `.ai/framework-guide.md`, list 2–3 **reference tests** your team considers good.
   This one step gives the biggest improvement.
4. For your next real task, copy `.ai/tasks/_TASK_TEMPLATE.md`, fill it in, and tell your agent:

   > Read AGENTS.md, then execute .ai/tasks/2026-10-21-booking-validation.md

5. Compare with how it went before — and [tell us](#try-it-and-report-back).

## What's inside

```
template/                         ← copy this into your repo
├── AGENTS.md
├── CLAUDE.md                     pointer to AGENTS.md (Claude Code)
├── .github/copilot-instructions.md   pointer + short rules (GitHub Copilot)
└── .ai/
    ├── qa-context.md
    ├── framework-guide.md
    └── tasks/
        ├── _TASK_TEMPLATE.md
        └── examples/             Playwright Test · TypeScript
            ├── 01-add-api-tests.md
            ├── 02-fix-flaky-test.md
            ├── 03-missing-requirements.md
            └── 04-automate-manual-cases.md   (additional example)
docs/
├── pattern.md                    the pattern, what goes where, how to keep it alive
└── comparison.md                 the same 3 tasks with vs. without the pattern + scoring rubric
```

The example tasks use **Playwright + TypeScript**. The template itself is stack-agnostic —
with Java/Selenide/REST Assured, C#/NUnit or Python/pytest only paths, commands and API names
change, not the structure. PRs adding example tasks for other stacks are welcome
(see [CONTRIBUTING.md](CONTRIBUTING.md)).

## With vs. without — in one table

*Illustrative — describes common failure modes and what the pattern targets. Recorded results are
pending; see [evidence status](docs/comparison.md#evidence-status).*

| Task | Typical result without | What the pattern changes |
|---|---|---|
| Add API validation tests | Raw HTTP calls, guessed rules, status-only asserts | Existing client + builder, rules from spec, asserts code **and** field |
| Fix a flaky UI test | Sleep / retry / higher timeout | Shortcuts forbidden, root cause explained, fix in the page object |
| Missing requirements | Rules the agent made up become assertions | Agent is instructed to name what's missing and ask first |
| *Additional:* automate manual cases | Selectors in tests, own login setup, faked hard case | Page objects, existing fixtures, blocked case reported honestly |

Most of these changes are designed to come from the **persistent** files, not the task file — so
even a one-line prompt carries them. That's the difference between writing better prompts and having a repeatable workflow.

Details, a persistent-vs-per-task breakdown and a scoring rubric for your own runs:
[docs/comparison.md](docs/comparison.md)

A recorded comparison on real agent output, with a mock API and injected bugs, lives in the
[sandbox repository](https://github.com/Yevhen-Bozhenko/qa-agent-context-kit-sandbox).

## Tool compatibility

| Tool | Reads |
|---|---|
| Tools that support the `AGENTS.md` convention | `AGENTS.md` directly |
| Claude Code | `CLAUDE.md` (imports `AGENTS.md`) |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Anything else | Start your prompt with *"Read AGENTS.md first"* |

Tool support for instruction files changes quickly — check your tool's documentation.
If your tool reads a different file, add a one-line pointer to `AGENTS.md` rather than duplicating content.

## Try it and report back

This repo is an experiment in public: does a shared structure actually make agents better at QA work?

1. Apply the template to a real project.
2. Run one task with and without it; score both with the [rubric](docs/comparison.md#score-your-own-runs).
3. Open an issue:
   - 📊 **[Field report](../../issues/new?template=field-report.yml)** — your results, good or bad
   - 💡 **[Template improvement](../../issues/new?template=template-improvement.yml)** — a section that's missing, unclear or useless
   - 🧩 **Example task** — a PR adding a task file from your domain to `template/.ai/tasks/examples/`

Negative results are just as useful as positive ones.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE) © 2026 Yevhen Bozhenko
