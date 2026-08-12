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