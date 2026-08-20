# Bug Log

## Finding 1: Inconsistent error codes across amount validation
**Severity:** Low (documentation/consistency issue, not a functional bug)
**Environment:** Stripe Sandbox, PaymentIntent creation endpoint

**Steps:**
1. Create a PaymentIntent with amount = 49 (one cent below USD minimum)
2. Create a PaymentIntent with amount = 0
3. Create a PaymentIntent with amount = -100
**Expected:** Consistent error code for all "invalid amount" scenarios
**Actual:** Amount 49 returns `amount_too_small`. Amounts of 0, negative,
and non-integer values all return the more generic `parameter_invalid_integer`,
despite some of their error messages also referencing the minimum charge
threshold.
**Impact:** Minor — could complicate client-side error handling if a
developer assumes one error code covers all amount-validation failures.

## Finding 2: Currency codes are case-normalized silently
**Severity:** Low (undocumented behavior, not a functional bug)
**Environment:** Stripe Sandbox, PaymentIntent creation endpoint
**Steps:** Create a PaymentIntent with currency = "USD" (uppercase)
**Expected:** Either rejected as invalid, or preserved as submitted
**Actual:** Silently normalized and stored as "usd" (lowercase), with no
indication in the response that normalization occurred.
**Impact:** Minor — worth knowing for any code that compares currency
values by exact string match.

## Finding 3: Authentication failures consistently return `invalid_request_error`, not `authentication_error`
**Severity:** Low (documentation/consistency issue, not a functional bug)
**Environment:** Stripe Sandbox, PaymentIntent creation endpoint
**Steps:**
1. Send a Create PaymentIntent request with no Authorization header at all
2. Send the same request with an invalid/garbage secret key
3. Send the same request using a publishable key (`pk_test_...`) instead of
   a secret key
4. Send the same request with a malformed header
**Expected:** Based on Stripe's general error-type documentation, which
lists a distinct `authentication_error` type separate from
`invalid_request_error`, authentication-related failures were expected to
return `type: "authentication_error"`.
**Actual:** All four cases returned `type: "invalid_request_error"`
instead. Status codes correctly distinguished the cases (401 for missing/
invalid key/malformed, 403 for publishable-key-used-as-secret with a specific
`error.code: "secret_key_required"`), but the `type` field did not reflect
an authentication-specific category in any of them.
**Impact:** Low-to-moderate — a client integration that routes or handles
errors specifically by checking `type === "authentication_error"` (a
pattern shown in Stripe's own official error-handling documentation across
multiple languages) would fail to catch any of these real authentication
failures through that check alone, and would need to rely on status code
or `error.code` instead.

Note
A malformed header was r=tried to be created using whitespace but postman normalized this.
