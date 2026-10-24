# Task: Diagnose and fix the flaky "apply discount code" UI test

<!-- Example task file. Paths and names are illustrative — adapt to your project. -->

## 1. Task

Find the root cause of the intermittent failure in `checkout > should apply a valid discount code`
and fix it in the test code or framework — without hiding the symptom.

## 2. Context

- Test: `tests/ui/checkout/discount.spec.ts`, test `should apply a valid discount code`.
- Fails ~1 in 10 CI runs, never reproduced locally on a fast machine.
- CI failure excerpt:

  ```
  Error: expect(locator).toHaveText(expected)
  Expected: "£90.00"
  Received: "£100.00"
  Locator: getByTestId('order-total')
  ```

- The discount is applied by an async call to `/api/cart/discount`; the total re-renders after it.

## 3. Constraints

- Allowed: `tests/ui/checkout/**`, `src/pages/CheckoutPage.ts`.
- Forbidden fixes: `waitForTimeout`, raising timeouts, `test.retry`/`retries`, `test.fixme`,
  loosening the expected value.
- Time-box: if you cannot identify a plausible root cause from code + logs, stop and report
  what you checked and what extra evidence (trace, video, network log) you need.

## 4. Examples

- Approved waiting pattern: `CartPage.removeItem()` in `src/pages/CartPage.ts` waits for the
  matching network response, then for the UI state — imitate it.
- Counter-example: `tests/legacy/checkout-old.spec.ts` uses fixed sleeps — do not copy it.

## 5. Expected output

1. Root-cause explanation in 3–5 sentences (what races with what).
2. The fix (diff), ideally in the page object so other tests benefit.
3. List of other tests that use the same pattern and may be affected (do not change them —
   just list them).

## 6. Validation

- Run the test 20 times: `npx playwright test tests/ui/checkout/discount.spec.ts --repeat-each=20`
  — report pass count.
- Also run with CPU throttling / slow network if the project config supports it: TODO command.
- Explain why the fix removes the race rather than making it less likely.
