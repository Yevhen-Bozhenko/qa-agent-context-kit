# With vs. without the pattern

Three typical QA cases, handled two ways — API validation tests, a flaky UI test, and
[missing requirements](#task-3--missing-requirements) — plus one
[additional example](#additional-example--automate-manual-test-cases). The examples use
Playwright + TypeScript, but the pattern is the same for any stack.
Each section states its evidence status; see [Evidence status](#evidence-status).

- **Without** — a typical one-off chat prompt, no persistent project context.
- **With** — the same request as a task file (`.ai/tasks/examples/`) on top of `AGENTS.md`,
  `.ai/qa-context.md` and `.ai/framework-guide.md`.

Each improvement below is marked by where it comes from:

- **Persistent** — written once in `AGENTS.md`, `qa-context.md` or `framework-guide.md`;
  applies to every task automatically.
- **Per task** — written in the task file for this task only.

The split matters. If every improvement came from the task file, the pattern would just be
"write better prompts" — still one-off prompting. Most of the gains below are **Persistent**:
that is the shift from one-off prompting to a repeatable workflow.

> **Honesty note.** The "typical result" lists describe failure modes that QA engineers
> commonly report when using coding agents on test code. They are qualitative, not a benchmark.
> A well-written one-off prompt can avoid some of them — once, if you remember to say everything.
> The code snippets are illustrative and shortened to show the difference; they are not recorded agent output.
> Measured results from real teams are exactly what the [field reports](../.github/ISSUE_TEMPLATE/field-report.yml)
> are for — use the [scoring rubric](#score-your-own-runs) below and share yours.

---

## Task 1 — Add API validation tests

> **Evidence:** illustrative below. Recorded runs of this scenario: **pending** — see
> [Recorded results](#recorded-results).

**Without the pattern**

> "Write negative tests for POST /bookings validation."

Typical result:

- Raw HTTP calls (`fetch` / raw `request`) instead of the existing `BookingsClient`.
- Hand-written payloads instead of the data builder → duplicated, brittle data.
- Assertions on status code only — a `400` for the *wrong* field still passes.
- Invented validation rules (the agent guesses the limits).
- Missing project tags / test-case links; new file in a random folder.

**With the pattern** — [`01-add-api-tests.md`](../template/.ai/tasks/examples/01-add-api-tests.md)

| What changes | Comes from | Source |
|---|---|---|
| Uses `BookingsClient` and the data builder | Framework guide — reuse order | Persistent |
| Same structure as existing API tests | Framework guide — reference tests | Persistent |
| Error tests assert code **and** field, not just status | Framework guide — assertions | Persistent |
| Right folder, tags, test-case link | QA context — conventions | Persistent |
| Unexpected response → reported as a possible bug, not "fixed" in the test | AGENTS.md — working rules | Persistent |
| Rules come from the spec, not guesses | Task — spec excerpt | Per task |
| Exact cases, including boundaries | Task — case table | Per task |

**In code** *(illustrative, shortened)*

```ts
// Without: raw HTTP, hand-written payload, status-only assertion
test('rejects invalid booking', async ({ request }) => {
  const res = await request.post('/api/v2/bookings', {
    data: { checkIn: '2026-11-10', checkOut: '2026-11-09', guests: 2 },
  });
  expect(res.status()).toBe(400); // still passes if the error is about the wrong field
});
```

```ts
// With: existing client + builder, cases from the spec, asserts code AND field
const cases: { name: string; patch: (b: BookingBuilder) => BookingBuilder; field: string }[] = [
  { name: 'checkOut before checkIn', patch: b => b.checkOutDaysAfterCheckIn(-1), field: 'checkOut' },
  { name: 'checkOut equals checkIn', patch: b => b.checkOutDaysAfterCheckIn(0), field: 'checkOut' },
  { name: 'guests = 0', patch: b => b.withGuests(0), field: 'guests' },
  { name: 'guests = 9', patch: b => b.withGuests(9), field: 'guests' },
  { name: 'checkIn missing', patch: b => b.withoutCheckIn(), field: 'checkIn' },
];

test.describe('POST /bookings validation',
  { tag: ['@api', '@regression'], annotation: { type: 'issue', description: 'BOOK-412' } }, () => {
  for (const c of cases) {
    test(`rejects booking when ${c.name}`, async ({ bookingsClient }) => {
      const res = await bookingsClient.create(c.patch(aBooking()).build());

      expect(res.status()).toBe(400);
      const body: ErrorResponse = await res.json();
      expect(body.code).toBe('VALIDATION_ERROR');
      expect(body.errors.map(e => e.field)).toEqual([c.field]);
    });
  }
});
```

---

## Task 2 — Fix a flaky UI test

> **Evidence:** illustrative only — not experimentally verified. This scenario is not part of
> the recorded experiment.

**Without the pattern**

> "This test is flaky, fix it." *(+ error message)*

Typical result:

- Adds `waitForTimeout(2000)`, bumps the timeout or enables retries → green, but the race is still there.
- Sometimes loosens the expected value.
- No explanation of the root cause; fix lives inside one test, other tests keep the bug.

**With the pattern** — [`02-fix-flaky-test.md`](../template/.ai/tasks/examples/02-fix-flaky-test.md)

| What changes | Comes from | Source |
|---|---|---|
| No sleeps, retries, raised timeouts or loosened assertions | AGENTS.md + QA context — never do | Persistent |
| Fix uses the team's approved waiting pattern | Framework guide — synchronisation | Persistent |
| Fix lives in the page object, so other tests benefit | Framework guide — layers | Persistent |
| Agent is pointed at the network → re-render race | Task — CI log, async call | Per task |
| Root cause written down + other affected tests listed | Task — expected output | Per task |
| Proven stable: `--repeat-each=20` + why the race is gone | Task — validation | Per task |

**In code** *(illustrative, shortened)*

```ts
// Without: green on most runs, the race is still there
await page.getByTestId('discount-code').fill('SAVE10');
await page.getByRole('button', { name: 'Apply' }).click();
await page.waitForTimeout(2000);                                   // added by the agent
await expect(page.getByTestId('order-total'))
  .toHaveText('£90.00', { timeout: 15_000 });                     // and the timeout raised
```

```ts
// With: wait for the real event, in the page object, so every test benefits
// src/pages/CheckoutPage.ts
async applyDiscount(code: string): Promise<Response> {
  await this.discountInput.fill(code);
  const [response] = await Promise.all([
    this.page.waitForResponse(r =>
      r.url().includes('/api/cart/discount') && r.request().method() === 'POST'),
    this.applyButton.click(),
  ]);
  return response; // page objects return values; tests assert
}

// tests/ui/checkout/discount.spec.ts
const response = await checkoutPage.applyDiscount('SAVE10');
expect(response.ok()).toBe(true);
await expect(checkoutPage.orderTotal).toHaveText('£90.00');
```

The code is only half of the result. The other half is the written root cause and the list of
other tests with the same pattern, which the task file asks for.

---

## Task 3 — Missing requirements

> **Evidence:** the example task file below is **illustrative**. Recorded runs of the
> missing-requirements scenario are **pending** — no outcome is claimed until they exist; see
> [Recorded results](#recorded-results).

**Task:** generate API validation tests for `POST /bookings` when the validation rules are
**not documented** — no ticket, no acceptance criteria, and a spec that describes only the
request shape.

> "Add negative tests for POST /bookings validation."

| | What to observe |
|---|---|
| **Without context** | Does the agent assert rules it was never given (e.g. a guest limit it picked itself), or does it ask what the rules are? |
| **With persistent context** | Does the agent follow the `AGENTS.md` rule *"Do not guess expected behaviour"* — identify that the requirements are missing and ask, or report the task as blocked? |

**Expected behaviour:** the agent names the missing requirements (which limits, which error
format) and asks for them **before** writing assertions. Tests built on invented rules can pass
against the current API and still encode a bug — or fail and get "fixed" by copying whatever the
API returns.

**Where it comes from:** `AGENTS.md` → working rules → *Do not guess expected behaviour*, and
`qa-context.md` → where requirements live. Both are **Persistent**, so they apply even to a
one-line prompt with no task file — which is exactly the situation the experiment tests.

**How it's measured:** per run — *asked for the rules?* and *invented rules (which)?*, recorded by
a human from the session transcript; then, once the rules are given, the same automated checks
as Task 1. Protocol: [sandbox `experiments/PROTOCOL.md`](https://github.com/Yevhen-Bozhenko/qa-agent-context-kit-sandbox/blob/experiment-v1/experiments/PROTOCOL.md).

**With a task file** — [`03-missing-requirements.md`](../template/.ai/tasks/examples/03-missing-requirements.md)
*(illustrative)*: when you already know the requirements are incomplete, say so in the task.
The example ticket specifies some rules and leaves others open; the expected output is tests for
the specified rules **plus a list of concrete open questions** for the rest — no assertions for
rules nobody wrote down.

| What changes | Comes from | Source |
|---|---|---|
| Doesn't turn guesses into assertions | AGENTS.md — don't guess expected behaviour | Persistent |
| Knows where requirements live | QA context — tickets and acceptance criteria | Persistent |
| Tests only the rules that are written down | Task — constraints | Per task |
| Concrete questions that say what they block | Task — examples, expected output | Per task |
| Every assertion traced to a criterion | Task — validation | Per task |

> This example task file is **not** used in the sandbox experiment: it tells the agent explicitly
> what to do, so it would decide the outcome the experiment is trying to observe.

---

## Additional example — Automate manual test cases

> **Additional material** — not one of the three webinar cases.
> **Evidence:** illustrative only — not experimentally verified.

**Without the pattern**

> "Automate these 4 test cases." *(+ pasted table)*

Typical result:

- Selectors and actions inline in the test body instead of page objects.
- Creates its own login/user setup instead of the existing fixture.
- TC-213 "automated" with a 30-minute sleep, or quietly faked.
- TC-211 asserts that a message is visible, not that it's *identical* — the security point is lost.

**With the pattern** — [`04-automate-manual-cases.md`](../template/.ai/tasks/examples/04-automate-manual-cases.md)

| What changes | Comes from | Source |
|---|---|---|
| New page objects, no selectors in test bodies | Framework guide — layers | Persistent |
| Reuses the existing user fixture and `MailboxClient` | Framework guide — reuse order | Persistent |
| Test-case IDs in tags, project naming | QA context — conventions | Persistent |
| Hard case reported as not done instead of faked | AGENTS.md — don't invent, report what wasn't verified | Persistent |
| Explicit rule for TC-213: do it properly or mark blocked | Task — constraints | Per task |
| TC-211 assertion that proves the messages are identical | Task — validation | Per task |
| TC → test mapping table | Task — expected output | Per task |

**In code** *(illustrative, shortened — TC-211 only)*

```ts
// Without: raw selectors in the test, hardcoded email, assertion proves nothing
test('TC-211 unknown email', async ({ page }) => {
  await page.goto('/forgot-password');
  await page.locator('input[name="email"]').fill('unknown@test.com');
  await page.locator('button[type="submit"]').click();
  await expect(page.locator('.alert-success')).toBeVisible(); // visible ≠ identical
});
```

```ts
// With: page object, existing fixture, assertion that proves the requirement
test('TC-211 unknown email gets the same message as a registered one',
  { tag: '@regression', annotation: { type: 'tms', description: 'TC-211' } },
  async ({ page, freshUser }) => {
    const forgotPassword = new ForgotPasswordPage(page);

    const registered = await forgotPassword.requestReset(freshUser.email);
    const unknown = await forgotPassword.requestReset(`unknown-${Date.now()}@example.test`);

    // Identical text, so the response can't reveal which accounts exist
    expect(unknown).toBe(registered);
  });
```

---

## Same short prompt, with and without persistent context

The task file is meant to be optional for most of the benefit. Take the one-line prompt from Task 1:

> "Add negative tests for POST /bookings validation."

*What each condition is designed to produce — the intent of the pattern, not recorded results.*

| | No context | Persistent context only | Persistent context + task file |
|---|---|---|---|
| Uses existing client, builder, fixtures | ❌ | ✅ | ✅ |
| Matches reference tests, folders, tags | ❌ | ✅ | ✅ |
| Error assertions check code and field | ❌ | ✅ | ✅ |
| No forbidden shortcuts | ❌ | ✅ | ✅ |
| Reports assumptions and what it ran | ❌ | ✅ | ✅ |
| Validation rules | ❌ guessed and asserted | ⚠️ asks for the spec, or reports blocked | ✅ taken from the spec |
| Exact cases and boundaries covered | ❌ | — (waits for the spec) | ✅ |

With persistent context, even a one-liner carries the framework, conventions and honesty rules
— including the instruction to ask, rather than invent, when a requirement is missing. Whether
agents actually follow these instructions is what the [recorded results](#recorded-results) test.
The task file is only needed for what is genuinely specific to this task:
the spec, the cases, the proof.

---

## Recorded results

The examples above are illustrative. A recorded comparison on real agent output lives in a
separate repository, so this kit stays a pure template:
**[qa-agent-context-kit-sandbox](https://github.com/Yevhen-Bozhenko/qa-agent-context-kit-sandbox)**.

It runs the Task 1 scenario (POST /bookings validation) from one pinned commit, with the same model,
prompt and clarification answers. **Every condition gets the same requirements** (the BOOK-412
ticket); only the context changes:

| Condition | What the agent gets | Answers |
|---|---|---|
| No context | Project + ticket + one-line prompt | baseline |
| Persistent context | + `AGENTS.md`, `qa-context.md`, `framework-guide.md` | Does persistent context improve framework adherence? |
| Full pattern | + task file | Do structured task instructions improve requirement coverage? |

A separate **missing-requirements** scenario removes the ticket: there, success means the agent
asks for the rules instead of inventing them.

Besides the rubric, each run's tests are re-run against three deliberately broken versions of the
API (same-day check-out accepted, error on the wrong field, 9 guests accepted). Tests that only
pass don't count — they have to catch the bugs.

What the comparisons can show — and what they can't: persistent context is compared at equal
requirements; the task file adds both structure and explicit content, so its effect is reported
as a whole, not as "structure alone".

*Results: pending. They will be published per run (not only averages), with links to the raw
agent output and checker reports.*

### Evidence status

| Section | Evidence |
|---|---|
| Task 1 — API validation tests | Illustrative here; recorded runs **pending** (sandbox, main scenario) |
| Task 2 — Flaky UI test | **Illustrative only** — not experimentally verified |
| Task 3 — Missing requirements | Example task file illustrative; recorded runs **pending** (sandbox, missing-requirements scenario, run without that task file) |
| Additional example — Manual test cases | **Illustrative only** — not experimentally verified |
| Same short prompt table | Design intent, not recorded results |

---

## Consistency across people and sessions

Without shared context, the same task given by two engineers — or by one engineer in two
sessions — produces two different styles of test, depending on how each prompt was worded.

With the pattern, both agents read the same `AGENTS.md`, `qa-context.md` and `framework-guide.md`,
so the results share structure, conventions and report format. And every correction a reviewer
makes twice becomes one line in `qa-context.md`, so it reaches everyone's next task — not only
the chat where it was made.

---

## The common thread

| Without | With |
|---|---|
| Agent fills gaps with plausible guesses | Gaps are filled by context the team wrote once |
| "Green" is the goal | "Proven correct" is the goal (Validation) |
| Each prompt starts from zero | Persistent files carry conventions across tasks |
| Results depend on who wrote the prompt | Results follow the same structure for everyone |
| Review finds style + correctness issues | Review focuses on correctness |
| Corrections are lost after the chat | Corrections become lines in `qa-context.md` |

---

## Score your own runs

Run the same task on your project and score each run. At minimum compare
**no context** vs. **full pattern**; if you can, also try **persistent context only**
with your usual short prompt — it shows how much the persistent files carry on their own.
Use the totals in a [field report](../../../issues/new?template=field-report.yml).

| # | Criterion | 0 | 1 | 2 |
|---|---|---|---|---|
| 1 | Compiles / type-checks first time | no | after small fixes | yes |
| 2 | Tests pass for the right reason | no | partly | yes |
| 3 | Reuses existing framework code | ignores it | partly | fully |
| 4 | Follows conventions (naming, tags, folders, style) | no | partly | yes |
| 5 | Assertions prove the requirement | status/visibility only | partly | yes |
| 6 | No forbidden shortcuts (sleeps, retries, weakened asserts) | several | one | none |
| 7 | Honest report (assumptions, what wasn't verified) | none | partial | complete |
| | **Number of follow-up corrections you had to give** | | | *count* |

Maximum score: 14. The correction count is often the most telling number.
