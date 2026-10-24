# QA project context

> Persistent context for AI coding agents. Fill every `TODO`, delete the hints you
> don't need. Keep it short and factual — this file is loaded on every task, so every
> line costs attention. Target: under ~200 lines.
>
> Maintenance rule: **every time you correct the agent on the same thing twice,
> add one line here.**

## 1. Product and domain

- What the system under test does: TODO (one or two sentences)
- Main user roles: TODO (e.g. admin, clinician, guest)
- Business rules the tests care about most: TODO
- Things that look like bugs but are intended behaviour: TODO

## 2. Test stack

| Item | Value |
|---|---|
| Language / version | TODO (e.g. TypeScript 5.x, Java 21) |
| Test runner | TODO (e.g. Playwright Test, JUnit 5, TestNG, pytest) |
| UI driver | TODO (e.g. Playwright, Selenide, none) |
| API client | TODO (e.g. Playwright `request`, REST Assured) |
| Assertions | TODO (e.g. `expect`, AssertJ, FluentAssertions) |
| Reporting | TODO (e.g. Allure, HTML reporter) |
| Build / package manager | TODO (e.g. npm, Maven, Gradle) |
| CI | TODO (e.g. GitHub Actions, Jenkins) |

Do not add new dependencies without asking.

## 3. Repository map

```
TODO — paste a trimmed tree, 2–3 levels deep, with one comment per folder, e.g.
tests/
  api/            # API tests, one file per resource
  ui/             # UI tests, grouped by feature
src/
  pages/          # page objects
  clients/        # API clients
  fixtures/       # test fixtures / setup
  data/           # test-data builders and static data
```

## 4. How to run

| Purpose | Command |
|---|---|
| All tests | TODO |
| One file | TODO |
| One test by name | TODO |
| By tag / group | TODO |
| Lint | TODO |
| Type check / compile | TODO |
| Open report | TODO |

Agents: run the narrowest command that covers your change, then lint + type check.

## 5. Environments and configuration

- Default environment for local runs: TODO
- How the base URL / env is selected: TODO (env var, config file, profile)
- Secrets: TODO (where they come from — never hardcode, never print)
- Environments agents must never target: TODO (e.g. production)

## 6. Test data and accounts

- How tests get users/accounts: TODO (fixture, account pool, API seeding)
- How tests create data: TODO (builders, factories, API calls)
- Cleanup rule: TODO (e.g. each test cleans up what it creates)
- Data that is shared and must not be modified: TODO

## 7. Conventions

- File naming: TODO
- Test naming: TODO (e.g. `should <outcome> when <condition>`)
- Tags / groups: TODO (e.g. `@smoke`, `@regression`, `@api`)
- Test structure: TODO (e.g. Arrange / Act / Assert; one behaviour per test)
- Locators: TODO (e.g. `data-testid` first, then role; no XPath by text)
- Waiting: TODO (e.g. rely on auto-waiting / explicit conditions; never hard waits)
- Assertions: TODO (e.g. assert on business outcome, not only status code)
- Reporting / steps: TODO (e.g. wrap meaningful actions in Allure steps)
- Comments: TODO (e.g. link the test case ID / ticket in a tag or annotation)

## 8. Definition of done for a test

A new or changed test is done when:

- [ ] It fails for the right reason when the behaviour is broken (checked or reasoned).
- [ ] It passes locally at least TODO times in a row.
- [ ] It is independent: no order dependency, no leftover data.
- [ ] It uses existing framework building blocks where they exist.
- [ ] Lint and type checks pass.
- [ ] TODO (team-specific: tags, test-case link, report steps…)

## 9. Never do

- Never weaken or delete assertions to make a test pass.
- Never add hard waits, blanket retries or inflated timeouts.
- Never commit secrets, tokens or real personal data.
- Never change production code unless the task says so.
- TODO (team-specific)

## 10. Known pitfalls

- TODO (e.g. "the grid renders rows lazily — scroll before counting")
- TODO (e.g. "the notifications service is eventually consistent — poll, don't sleep")
- TODO (e.g. "tests tagged @quarantine are known flaky — don't use them as references")

## 11. Glossary

| Term | Meaning |
|---|---|
| TODO | TODO |
