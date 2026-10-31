# The pattern

The goal is to move from one-off prompting to a repeatable workflow:
the same kind of task should get the same quality of result, regardless of who
asks or how well they phrase it that day.

## Three layers

| Layer | File | Changes | Answers |
|---|---|---|---|
| **Entry point** | `AGENTS.md` | almost never | What to read, working rules, how to report |
| **Persistent project context** | `.ai/qa-context.md` | when the project changes | What is this project and how do we test it? |
| **Framework guide** | `.ai/framework-guide.md` | when the framework changes | How do I extend *this* framework correctly? |
| **Task instruction** | `.ai/tasks/*.md` | every task | What exactly do you want, and how will we know it's right? |

### What goes where

Ask: *would I have to repeat this for the next task?*

| Information | Goes in |
|---|---|
| Test runner, commands, folder layout | `qa-context.md` |
| "Never use hard waits" | `qa-context.md` (Never do) |
| "Use `data-testid` locators" | `qa-context.md` (Conventions) |
| "Page objects have no assertions" | `framework-guide.md` |
| "Model API tests on `users.spec.ts`" | `framework-guide.md` (reference tests) |
| Ticket, spec excerpt, error log | task file (Context) |
| "Only touch `tests/api/bookings/`" | task file (Constraints) |
| The list of cases to cover | task file (Examples) |
| "Run it 20 times" | task file (Validation) |

## The task structure

**Task → Context → Constraints → Examples → Expected Output → Validation**

| Section | Purpose | Most common mistake |
|---|---|---|
| Task | One outcome, stated plainly | Several unrelated outcomes in one task |
| Context | Facts the agent can't discover itself | Pasting everything instead of the relevant excerpt |
| Constraints | Scope + forbidden shortcuts | Leaving out the shortcuts you *know* it will take |
| Examples | Something concrete to imitate | Describing style in words instead of pointing at a file |
| Expected output | Shape of the deliverable | "Write some tests" — no names, count or location |
| Validation | How the agent proves it | "Make sure it passes" — passing is not proof |

Validation is the section that matters most for QA. An agent optimises for the
finish line you give it; if the finish line is "green", you will get green.
Make the finish line "proven correct": a negative check, a stability run,
a rule about what to do when reality disagrees with the spec.

## Keeping it alive

- **Two-strikes rule:** if you correct the agent on the same thing twice, add a line
  to `qa-context.md` or `framework-guide.md`.
- **Ask the agent:** `AGENTS.md` asks it to suggest context additions at the end of each task.
  Review them like code.
- **Prune:** if a line hasn't mattered in months, remove it. Long context dilutes attention.
- **Review in PRs:** changes to `.ai/` and `AGENTS.md` go through code review like any other code.
- **Reusable tasks:** a task file you write more than twice (e.g. "triage a failed nightly run")
  becomes a template in `.ai/tasks/` with only the variable parts left blank.

## Anti-patterns

- **The wiki dump** — pasting the whole Confluence space into the context file.
- **Stale context** — the file says Selenium 3, the repo uses Playwright.
- **Wishful conventions** — documenting how you'd *like* the code to look, while every
  reference file does something else. The agent follows the code, not the wish.
- **Rules without examples** — "follow our page object pattern" with no file to look at.
- **Validation by vibes** — no command, no count, no negative check.

## Why it works

The kit isn't a new technique. It applies well-known prompt and context practices once, in files,
so nobody has to remember them for each task.

| Practice | Where the kit applies it |
|---|---|
| Be clear about what "done" looks like | Task file → *Task*, *Expected output*, *Validation* |
| Give context and the reason | Task file → *Context*; `qa-context.md` |
| Show examples | Reference tests in `framework-guide.md` |
| Specify the output format | Task file → *Expected output*; report format in `AGENTS.md` |
| Separate instructions from data | Six fixed sections in every task file |
| One task at a time | One outcome per task file |
| Keep memory outside the chat | `AGENTS.md`, `qa-context.md`, `framework-guide.md` |
| Select only what's relevant | Excerpts in *Context*, not whole documents (see *The wiki dump*) |
| Stable instructions first, task last | `AGENTS.md` → persistent files → task file |
| Keep facts consistent | *Prune*, *Stale context*, one fact in one place |
| Iterate and measure | Two-strikes rule, the [rubric](comparison.md#score-your-own-runs), the recorded results |
