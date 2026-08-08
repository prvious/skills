---
name: prvious-integration-development
description: Use this skill when designing, implementing, reviewing, or testing Laravel boundaries around third-party APIs, SDKs, webhooks, and remote systems, or when deciding whether provider-specific code needs direct SDK usage, a concrete adapter, or an application-owned gateway. Covers boundary selection, provider-type isolation, failure classification, remote-write safety, first-class fakes, and layered integration testing without imposing a reliability architecture.
---

# Laravel Integration Development

Build the smallest explicit boundary that keeps third-party behavior understandable without wrapping every library or inventing infrastructure the application has not adopted.

## Confirm the Scope

Apply this skill when code crosses a process or provider boundary, including payment providers, ERP systems, cloud APIs, email providers, external HTTP APIs, webhooks, and vendor SDKs that perform remote operations.

Do not apply it merely because code imports a third-party package. Use stable, well-maintained local utility libraries directly when no remote boundary, provider-specific failure model, or application contract needs isolation.

## Coordinate Areas of Authority

- `prvious-action-development` owns complete application use cases and their public contracts.
- This skill owns the boundary between application code and a third-party or remote system.
- When `Prvious\Result` is used, activate `prvious-result-development`; that package-owned skill and installed source own the exact Result API.
- Laravel Boost and project instructions remain authoritative for framework and package APIs, HTTP clients, queues, configuration, service providers, testing tools, and security mechanics.

Do not introduce the Prvious Action Pattern or Result solely because this integration skill activated. When the application already uses that pattern, express definitive caller-actionable rejections through its required `prvious/result` contract.

## Follow the Workflow

1. Inspect project instructions, installed package versions, provider documentation, nearby integrations, and existing testing facilities.
2. Identify the remote capability and the application code that consumes it.
3. Choose the smallest boundary: direct library use, a concrete provider adapter, or an application-owned interface with one or more adapters.
4. Define application-owned inputs and outputs only where they protect a meaningful boundary.
5. Classify each provider outcome as success, definitive expected rejection, operational failure, unknown remote outcome, or programming/configuration failure.
6. Identify retry, idempotency, transaction, webhook, and consistency decisions without selecting new infrastructure implicitly.
7. When the application owns an integration interface, provide a deterministic Fake and use it in consumer tests. Test the provider adapter separately through a provider-supported SDK fake or controlled transport, and prevent real remote requests from escaping tests.

## Preserve These Invariants

- Do not create an interface or wrapper that only mirrors a stable library without adding an application boundary.
- When the application owns an integration interface, provide a purpose-built Fake in test support and use it instead of a dynamic mock in application-consumer tests.
- Make Fake behavior explicit and application-level. Record requests, require configured outcomes, and never call the network, expose vendor types, simulate provider internals, or share mutable state between tests.
- Do not introduce an interface solely to obtain a Fake. Test an intentionally concrete adapter through the provider SDK's supported fake or a controlled transport seam.
- Keep provider-specific translation inside the provider-specific implementation once an application-owned boundary exists.
- Do not expose vendor response objects, status codes, or exception classes through an application-owned contract by accident.
- Translate only provider outcomes that are understood definitively. Preserve the original exception as `previous` when translating operational exceptions.
- Treat known caller-actionable rejection differently from outage, timeout, invalid configuration, malformed application requests, and unknown remote outcomes.
- Never catch `Throwable` merely to return a generic expected error.
- Never assume a timeout proves that a remote write did not happen, and never blindly retry a remote write.
- A local database transaction cannot roll back a remote side effect.
- Do not introduce an outbox, pending-attempt model, compensation workflow, reconciliation process, retry policy, or idempotency mechanism unless the project already uses it or the architecture decision is explicitly in scope.
- Verify webhook authenticity before trusting payloads and consider duplicate or out-of-order delivery according to the provider's guarantees.
- Prevent unfaked remote requests from escaping the ordinary test suite.

## Read the Relevant References

- Read [choosing-the-boundary.md](references/choosing-the-boundary.md) when deciding between direct library use, a concrete adapter, an interface, a service provider, an Action, or a Job.
- Read [failure-classification.md](references/failure-classification.md) whenever mapping provider responses or exceptions into application outcomes.
- Read [remote-operations.md](references/remote-operations.md) before remote writes, retries, webhooks, or work that mixes database state with provider effects.
- Read [testing.md](references/testing.md) whenever implementing or reviewing integration tests and fakes.
