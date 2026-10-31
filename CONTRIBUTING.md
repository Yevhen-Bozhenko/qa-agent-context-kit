# Contributing

Contributions that come from real use are the most valuable ones.

## Ways to contribute

- **Field report** — open an issue with the *Field report* template. Results with the
  scoring rubric from [docs/comparison.md](docs/comparison.md#score-your-own-runs) are especially useful.
- **Template improvement** — open an issue with the *Template improvement* template, or a PR.
- **Example task** — add a filled task file from your domain to
  `template/.ai/tasks/examples/`. Name it `NN-short-slug.md` and follow `_TASK_TEMPLATE.md`.
- **Another stack** — the examples use Playwright + TypeScript. Versions of the same example tasks
  for other stacks (Java, C#, Python…) are welcome under `template/.ai/tasks/examples/<stack>/`.
- **Tool pointer file** — if your agent reads a different instruction file, add a minimal
  pointer to `AGENTS.md` under `template/` and a row to the compatibility table in the README.

## Guidelines

- Keep the template **short**. Every line in a context file costs the agent attention;
  a new line should earn its place by preventing a real, repeated mistake.
- Keep it **stack-agnostic**. Stack-specific advice goes into examples, not the template.
- **Remove anything confidential** from examples and reports: company names, internal URLs,
  credentials, personal data.
- Prefer showing over telling: a concrete example beats a paragraph of rules.
- **Say what to do, not only what to avoid.** A "never" rule should name the alternative:
  "no hard waits — wait for the response or the element state".
- **One fact, one place.** When something changes, update the existing line instead of adding
  a new one. Pointer files (`CLAUDE.md`, `copilot-instructions.md`) repeat only the
  non-negotiables, and must change together with `AGENTS.md`.
- **Measure template changes.** Run a fixed task before and after the change and score both with
  the [rubric](docs/comparison.md#score-your-own-runs). "Sounds better" is not evidence.
