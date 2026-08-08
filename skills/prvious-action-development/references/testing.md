# Testing Actions

Use the project's established Laravel and testing conventions. Test the Action contract and durable effects rather than repeatedly testing `prvious/result` internals.

## Cover the Contract

Test every applicable behavior:

- The successful path and exact successful mutations.
- Every meaningful expected rejection.
- No unintended mutations on rejected paths for an atomic Action.
- Accurate durable progress and outcome reporting for an intentionally staged or best-effort Action.
- Boundary validation for user-facing uniqueness and referenced-record feedback when applicable.
- Ownership, tenancy, and cross-tenant rejection through the same scoped access used by the Action.
- Additional policy or domain authorization when the relationship alone is insufficient.
- Unexpected exception propagation.
- Local transaction rollback when an exception escapes.
- Parent-to-child error translation.
- Race-sensitive invariants through the adopted lock or atomic persistence operation.

Assert the Result variant before accessing its payload, following `prvious-result-development`. Assert the typed error and its context, not a magic string or incidental presentation message.

Do not define a raw `QueryException` as the expected contract for an ordinary duplicate or missing reference. Test the boundary's validation response and the Action's typed outcome. Test schema details separately only when the project has an established reason to verify its migrations; do not turn database exception behavior into an Action test.

## Cover Each Boundary

- Test controller mappings from every Action outcome to the intended HTTP response.
- Test that jobs handle expected errors explicitly.
- Test that retryable exceptions escape jobs.
- Test command exit behavior and listener reactions when they contain meaningful mapping.
- Test AI tool input validation, authentication, permission, and outcome mapping without asking a model to reproduce deterministic Action behavior.

## Test Composition

For nested atomic Actions, prove that child errors become the correct parent error and that rejected paths leave no unintended partial writes. For staged or best-effort workflows, prove that every committed step is represented honestly in durable state and the public outcome. Activate `prvious-integration-development` for tests involving third-party boundaries, provider adapters, remote failures, webhooks, retries, or idempotency.

Run the project's static analyzer whenever an Action input, output, or Result generic changes. Avoid duplicating package-level tests for Result factories, mapping, matching, narrowing, composition, extraction, or `Panic`; those behaviors belong to `prvious/result`.
