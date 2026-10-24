# Task: Add API tests for POST /bookings validation

<!-- Example task file. Paths and names are illustrative — adapt to your project. -->

## 1. Task

Add negative API tests for `POST /api/v2/bookings` covering the request validation rules listed below.

## 2. Context

- Ticket: BOOK-412 — validation was added for date range and guest count.
- API spec excerpt (from `docs/openapi.yaml`, `CreateBookingRequest`):
  - `checkIn`, `checkOut`: ISO date, required; `checkOut` must be after `checkIn`
  - `guests`: integer 1–8, required
  - On violation: `400` with body `{ "code": "VALIDATION_ERROR", "errors": [{ "field": "...", "message": "..." }] }`
- Existing happy-path tests: `tests/api/bookings/create-booking.spec.ts`
- Client: `src/clients/BookingsClient.ts` already has `create(request)`.

## 3. Constraints

- Only add or change files under `tests/api/bookings/` and, if needed, `src/data/bookings/`.
- Do not change `BookingsClient` signatures; adding an optional method is OK if justified.
- No new dependencies.

## 4. Examples

- Model the structure on `tests/api/users/create-user-validation.spec.ts`
  (parameterised cases, one `test()` per rule, shared valid payload from the builder).
- Cases to cover:

| Case | Payload change | Expected `errors[].field` |
|---|---|---|
| checkOut before checkIn | checkOut = checkIn − 1 day | `checkOut` |
| checkOut equals checkIn | checkOut = checkIn | `checkOut` |
| guests = 0 | guests: 0 | `guests` |
| guests = 9 | guests: 9 | `guests` |
| missing checkIn | omit checkIn | `checkIn` |

- Boundary sanity: `guests = 1` and `guests = 8` must be accepted (201).

## 5. Expected output

- New file `tests/api/bookings/create-booking-validation.spec.ts`.
- Tests tagged `@api @regression`, linked to `BOOK-412`.
- Each negative test asserts status **and** `code` **and** the offending `field`.

## 6. Validation

- Run: `npx playwright test tests/api/bookings/create-booking-validation.spec.ts` — all pass.
- Run lint and type check from `.ai/qa-context.md`.
- Negative check: explain which assertion would fail if the API returned `400`
  for the wrong field.
- If any case returns something other than the spec says, **do not adapt the test to
  the actual response** — stop and report it as a possible bug with the response body.
