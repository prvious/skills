# Remote Operations

Remote calls create failure windows that a local database transaction cannot close. Identify the relevant decisions without silently choosing a new reliability architecture.

## Ask Before a Remote Write

Determine:

1. Whether the operation is a read or a side-effectful write.
2. Whether the provider supports idempotency and how it defines duplicate requests.
3. Whether a timeout can leave the remote outcome unknown.
4. Whether the caller may retry and which failures are actually transient.
5. Which local state must exist before and after the request.
6. Whether the project already uses pending states, reconciliation, compensation, durable handoff, or another coordination pattern.

Do not convert these questions into infrastructure automatically. Follow an established project design. When none exists and correctness depends on the answer, surface the architecture decision before implementing an outbox, attempt table, scheduled reconciler, compensation workflow, or custom retry system.

## Keep Local and Remote Guarantees Honest

- Do not claim that a database rollback reverses a payment, message, webhook, provisioning request, or other remote effect.
- Avoid holding a database transaction open during a remote request unless the project has deliberately accepted that tradeoff.
- Do not treat a timeout as a definitive rejection or proof that nothing happened.
- Retry only failures classified as safe under the provider's guarantees and the project's policy.
- Reuse an established idempotency identity across retries when the adopted design requires one.

Activate `prvious-action-development` for local Action transaction ownership. This skill owns only the remote-boundary analysis and the integration behavior selected by the project.

## Handle Webhooks as Untrusted Delivery

Verify the provider signature or equivalent authenticity mechanism before trusting a webhook payload. Inspect the provider's guarantees for duplicate delivery, ordering, retries, event identifiers, and response deadlines.

Let the webhook boundary parse and authenticate delivery, then delegate meaningful application state transitions to an Action or the project's established use-case boundary. Do not scatter business mutations across webhook controllers.

Only add event persistence, deduplication, reordering, or reconciliation mechanisms when required by the provider contract and selected application architecture.
