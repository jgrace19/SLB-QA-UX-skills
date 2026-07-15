# Coverage Categories Checklist

For each testable requirement, check whether the categories below are addressed. Not every category applies to every requirement — use judgment and skip the ones that clearly don't fit.

## Functional
- **Happy path**: the primary success scenario works end to end.
- **Alternate paths**: valid secondary flows and optional branches.
- **Negative / error handling**: invalid input, failed dependencies, timeouts, and correct error messaging.
- **Boundary / edge cases**: empty, min, max, off-by-one, zero, very large, duplicates, concurrent actions.

## Data
- **Validation**: required fields, formats, ranges, type coercion.
- **State transitions**: each allowed transition, plus rejected illegal transitions.
- **Persistence / idempotency**: retries and repeated operations don't corrupt state.

## Access and security
- **AuthN/AuthZ**: each role/permission level, including unauthorized access is blocked.
- **Input safety**: injection, XSS, unsafe file/URL handling where relevant.

## Non-functional
- **Performance / load**: response under expected and peak volume.
- **Accessibility**: keyboard nav, screen readers, contrast (for UI work).
- **Compatibility**: browsers, devices, locales/timezones as applicable.
- **Observability**: logging/metrics exist to diagnose failures.

## Regression
- **Adjacent behavior**: existing features touched by the change still pass.
