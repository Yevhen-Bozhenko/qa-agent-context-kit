# Working with the existing test framework

> How AI agents should extend *this* framework instead of writing generic tests next to it.
> Fill the `TODO`s. The single most valuable thing in this file is section 1:
> concrete reference tests the agent can imitate.

## 1. Reference tests (golden examples)

Imitate these. They represent the current team standard.

| Kind of test | Reference file | Why it is a good example |
|---|---|---|
| API test | TODO `path/to/file` | TODO |
| UI test | TODO `path/to/file` | TODO |
| Data-driven / parameterised | TODO `path/to/file` | TODO |
| Page object / screen model | TODO `path/to/file` | TODO |
| API client | TODO `path/to/file` | TODO |

**Do not** use these as references (legacy, quarantined or about to be rewritten):
TODO `path/to/old/tests/**`

## 2. Layers

```
tests            → describe behaviour; no raw selectors, no raw HTTP
  ↓ use
flows / steps    → multi-step business actions (optional layer)  TODO: exists? where?
  ↓ use
page objects     → one per page/component; locators + actions, no assertions  TODO: confirm rule
API clients      → one per resource; typed request/response models
  ↓ use
core             → config, auth, http, logging, reporting
```

Rule: a test must not skip a layer (no selectors or raw HTTP calls inside test bodies).
TODO: adjust to your real architecture.

## 3. Reuse-first lookup order

Before creating anything new, search in this order and list what you found:

1. TODO `src/fixtures/` — is there a fixture that already gives you this user/state/page?
2. TODO `src/pages/` or `src/clients/` — does a page object / client method already exist?
3. TODO `src/data/` — is there a builder/factory for this entity?
4. TODO `src/utils/` — is there a helper for this wait/format/conversion?

Only create something new if nothing fits. If something *almost* fits, prefer
extending it (new method / optional parameter) over a near-duplicate.

## 4. How to add…

**A new page object**
- Location and naming: TODO
- Base class / constructor pattern: TODO
- Locator strategy: TODO
- Expose actions as methods; return values or next page objects; no assertions inside.

**A new API client method**
- Location and naming: TODO
- Request/response models: TODO (where they live, how they're generated)
- Auth: TODO (how the client gets a token)

**A new fixture / setup step**
- TODO (where fixtures are registered, scope rules, cleanup)

**New test data**
- TODO (builder/factory pattern, how to make unique values, cleanup)

## 5. Synchronisation and waiting

- TODO: what the framework already waits for automatically.
- TODO: the approved way to wait for a condition (helper name / API).
- Polling for eventually-consistent backends: TODO helper.
- Forbidden: fixed sleeps, increasing global timeouts, retries as a fix for flakiness.

## 6. Assertions

- Prefer assertions on the business outcome (state, data, visible result) over status codes alone.
- Prefer one behaviour per test; multiple assertions about that behaviour are fine.
- Use soft assertions only for TODO.
- Error-path tests must assert the error *content* (code, message, field), not just "it failed".

## 7. Reporting

- TODO (e.g. Allure: `@step` on page-object actions, `test.step()` for business steps,
  attach request/response on API failures, link test-case IDs).

## 8. When the framework is missing something

Do not silently invent a parallel mini-framework inside the test.
Instead: implement the smallest reusable addition in the right layer, and call it out
in your report so a human can review the design.

## 9. Self-check before you report back

- [ ] Every new file is in the folder this guide says it belongs to.
- [ ] No selectors or raw HTTP inside test bodies.
- [ ] Reused: list. Created new: list (with reason).
- [ ] No hard waits, no weakened assertions, no new dependencies.
- [ ] Ran: narrowest test command + lint + type check. Results included.
