# Transactions and Composition

Make the top-level orchestration Action own the transaction spanning the local database mutations that must succeed or fail together.

## Preserve Local Transaction Safety

- Use `DB::transaction()` directly.
- Check stable expected rejection paths before the first mutation whenever possible.
- Recheck mutable, race-sensitive state under a lock or enforce it through an established atomic persistence operation. Keep schema constraints as the final integrity safeguard, not the application's normal branching mechanism.
- Let unexpected exceptions escape the transaction closure so Laravel rolls back.
- Remember that returning an error Result from the closure is a normal return and therefore commits.
- Do not catch `Throwable` inside a transaction merely to convert it into an expected error.
- Do not return an error Result after unintended partial local writes.
- Let the parent own a transaction spanning several coordinated child mutations.

Structure a mutating transaction in this order:

1. Load the intended target through the appropriate actor or tenant scope.
2. Lock or atomically protect state that may change concurrently.
3. Return expected rejections before any write.
4. Perform the coordinated local mutations.
5. Return the successful Action value.

Do not catch `QueryException` or another broad database exception as normal Action control flow. Let unexpected persistence failures escape. When a concurrent conflict must be represented as an expected outcome, follow an established atomic Laravel or project persistence operation instead of adding ad hoc exception catches to Actions.

## Choose Atomic or Intentional Partial Progress

Define the use-case contract before choosing transaction boundaries:

- **Atomic:** every local mutation must succeed together. Use one owning transaction and ensure expected rejections occur before writes.
- **Staged:** intermediate state is deliberately durable. Give each stage an explicit state and transaction boundary, and make later recovery or continuation part of the modeled workflow.
- **Best effort:** independent items may succeed or fail separately. Give each item an appropriate transaction boundary and return or record an honest outcome report.

Do not add a vague "recovery path" exception to the atomic rule. If partial progress is intentionally committed, do not wrap the entire workflow in one transaction and do not return a generic error that conceals already-committed work. The public contract and durable state must make that progress observable.

## Compose Child Actions Intentionally

Before calling a mutating child Action, determine:

1. Which Action owns the local transaction.
2. Whether the child can return an expected rejection after writing.
3. Whether an earlier parent write would commit if the child returns that rejection.
4. How child errors translate into the parent's public contract.

Use Result composition only when it keeps those answers obvious. Prefer ordinary branching when outcomes affect transaction boundaries, logging, or further side effects. Follow `prvious-result-development` for the exact composition API.

Refactor unsafe children into precondition checks and mutation steps, or move their coordinated mutations under the parent transaction. Do not invent a transaction trait, base Action, or universal exception-to-Result helper.

## Route Remote Effects Elsewhere

A local database transaction does not make a third-party or remote side effect atomic. When a use case crosses that boundary, activate `prvious-integration-development` before selecting an approach. Follow an architecture already adopted by the project or surface the missing decision; do not introduce an outbox, compensation workflow, reconciliation process, retry policy, or idempotency mechanism from this Action skill alone.
