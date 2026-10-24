# Task: Automate manual test cases TC-210..TC-213 (password reset)

<!-- Example task file. Paths and names are illustrative — adapt to your project. -->

## 1. Task

Automate the four manual test cases below as UI tests in the existing framework.

## 2. Context

Manual cases (exported from the test management tool):

| ID | Title | Steps (short) | Expected |
|---|---|---|---|
| TC-210 | Reset link sent for registered email | Forgot password → enter registered email → submit | Confirmation message shown; email received |
| TC-211 | Same message for unknown email | Same, with unregistered email | Same confirmation message (no account enumeration) |
| TC-212 | Reset with valid link | Open link → set new valid password | Can log in with new password |
| TC-213 | Expired link rejected | Open link older than 30 min | "Link expired" message, password unchanged |

- Emails are captured by the test mail inbox; helper: `src/clients/MailboxClient.ts`.
- Users come from the `freshUser` fixture (`src/fixtures/users.ts`).
- There is **no** existing page object for the reset-password pages.

## 3. Constraints

- Do not use real email addresses.
- TC-213: do not wait 30 minutes. If there is no supported way to create an expired token
  (check `src/clients/TestSupportClient.ts`), automate TC-210..212 only and report TC-213 as blocked
  with the reason.
- Shared fixtures must not be modified; add a new fixture only if necessary.

## 4. Examples

- Page object style: `src/pages/LoginPage.ts`.
- Test style with mailbox: `tests/ui/registration/verify-email.spec.ts`.

## 5. Expected output

- `src/pages/ForgotPasswordPage.ts`, `src/pages/ResetPasswordPage.ts` following the page object rules.
- `tests/ui/auth/password-reset.spec.ts`, one test per TC, test-case ID in the annotation/tag.
- A table in your report mapping each TC to the test name and its status (automated / blocked + reason).

## 6. Validation

- Run the new spec 3 times — all pass.
- For TC-211, state which assertion proves the messages are identical (not just "both visible").
- Lint and type check pass.
