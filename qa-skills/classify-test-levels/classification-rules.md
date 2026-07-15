# Test Level Classification Rules

Assign each test case to the lowest level that can meaningfully verify the behavior. Use the signals below; when a case spans levels, pick the dominant intent and note the overlap.

## Unit
**Verifies:** a single function, method, class, or module in isolation.

Signals:
- Logic, calculations, validation, parsing, formatting, state machines.
- All external dependencies (DB, network, filesystem, clock, other services) are mocked/stubbed.
- Fast (ms), deterministic, no environment setup.

Tie-breaker: if the behavior can be checked by feeding inputs and asserting outputs with no real I/O, it's a unit test — even if it currently runs as manual.

## Integration
**Verifies:** two or more components working together within a bounded scope.

Signals:
- Service ↔ database, repository ↔ real schema, API endpoint ↔ handlers/middleware.
- Contract/boundary checks between modules or between your code and a real dependency (queue, cache, third-party client against a sandbox).
- Real dependencies but not the full deployed system; may need a test DB or containers.

Tie-breaker: crosses one component boundary with a real dependency but does not drive the whole app through its user entry point → integration.

## End-to-End (E2E)
**Verifies:** a complete user-facing workflow across the deployed stack.

Signals:
- Drives the real UI or public API from the outside (browser automation, HTTP against a running environment).
- Touches the full path: UI → API → services → data.
- Represents a user journey (sign up, checkout, submit-and-confirm).

Tie-breaker: exercises multiple layers through the app's real entry point and can be automated → E2E (not manual).

## Manual
**Verifies:** things requiring human judgment or impractical to automate.

Signals:
- Exploratory testing, usability, visual/design review, content correctness.
- Hardware, physical devices, or external systems with no automatable interface.
- One-off setup/verification, or flows not yet automatable.

Tie-breaker: only mark manual when automation is genuinely infeasible or not valuable — not merely because a case is currently performed by hand.

## Re-leveling flags

Call out common misplacements:
- Pure logic checks run as manual or E2E → should be **unit**.
- Broad UI E2E used to verify a single API rule → move down to **integration** or **unit**.
- "Integration" cases that mock everything → actually **unit**.
- Manual cases that are deterministic and scriptable → automate at the appropriate level.
