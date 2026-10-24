# Task: Add validation tests for PATCH /bookings/{id} — requirements incomplete

<!-- Example task file. Paths and names are illustrative — adapt to your project.
     Shows how to hand an agent a task whose business rules are only partly known:
     the deliverable is tests for what IS specified, plus questions for what isn't.
     Illustrative example — not used in the recorded sandbox experiment. -->

## 1. Task

Add validation tests for `PATCH /api/v2/bookings/{id}` (change an existing booking) — **only for
rules that are specified**. For everything else, list the open questions instead of writing tests.

## 2. Context

- Ticket BOOK-430 — "Guests can change their booking". Acceptance criteria as written today:
  1. `checkIn`, `checkOut` and `guests` can be changed; the same validation as on create applies
     (see BOOK-412: dates in `YYYY-MM-DD`, `checkOut` after `checkIn`, `guests` 1–8).
  2. "Changes are not allowed too close to the check-in date." — **no cut-off is given.**
  3. "Cancelled bookings can't be changed." — **no response code or error format is given.**
- The product owner is on leave until Thursday; the ticket is the only written source.
- Client: `BookingsClient` has `create()` and `get()`, but no `update()` yet.

## 3. Constraints

- Assert only rules that are written in BOOK-430 or BOOK-412. **Do not** infer a cut-off, status
  code or error format from the current API, existing tests or similar endpoints.
- If a rule is missing or ambiguous, do not write a test for it — not even a skipped one with
  guessed values. Put it in the questions list.
- You may add `BookingsClient.update()`; no other framework changes.

## 4. Examples

- Model specified-rule tests on `tests/api/bookings/create-booking-validation.spec.ts`.
- What a good open question looks like — concrete, answerable, says what it blocks:

  > **Q2 (blocks: change cut-off tests):** What is the minimum time before `checkIn` when a
  > booking can still be changed — 24 h, 48 h, or something else? Is it measured in the hotel's
  > time zone or UTC? What response is expected when it's too late (status, `code`, `field`)?

- Not a good question: "What are the rules for changes?" — too vague to answer or act on.

## 5. Expected output

1. `tests/api/bookings/update-booking-validation.spec.ts` covering criterion 1 only
   (date order, guest limits, date format — the rules BOOK-412 defines).
2. `BookingsClient.update(id, body)` if you needed it.
3. In your report, a numbered **open questions** list: one question per missing rule, each saying
   which tests it blocks. Expect at least criteria 2 and 3 to appear there.
4. No assertions for criteria 2 and 3.

## 6. Validation

- Run the new spec 3 times — all pass. Run lint and type check.
- For every assertion, name the acceptance criterion it comes from (e.g. "BOOK-412 #3").
  Any assertion you can't trace to a written criterion must be removed and turned into a question.
- If the API's current behaviour contradicts a written criterion, stop and report it as a
  possible bug — don't adapt the test.
