---
name: prvious-action-development
description: Use this skill when creating, changing, reviewing, or testing Laravel Actions in applications that use the Prvious Laravel Action Pattern, or when extracting a complete application use case from a controller, service, job, command, or listener. Apply it to code in `app/Actions`, Action `handle()` methods, and code that calls Actions. Requires `prvious/result` and covers typed inputs and outputs, business invariants, expected outcomes, exception semantics, local transaction ownership, nested composition, and caller-specific outcome handling.
---

# Laravel Action Development

Implement complete application use cases as typed Actions. Use `prvious/result` for every expected, caller-actionable rejection, but keep `void` or plain-value returns when an Action has no expected rejection.

## Confirm the Pattern and Its Dependency

Apply the Prvious Laravel Action Pattern when at least one condition is true:

- The user explicitly requests it or asks to adopt it.
- A project guideline or path-scoped rule declares it.
- Nearby Actions already use this pattern consistently.

The pattern requires `prvious/result`. Before implementing or changing an Action:

1. Verify that `prvious/result` is declared and installed through Composer.
2. Activate the package-owned `prvious-result-development` skill before working with any Result contract or API.
3. If the package skill is unavailable, read it from the installed package and inspect the installed source rather than guessing the API.
4. For implementation or adoption work, install a compatible package version when it is missing. For read-only review, report the missing prerequisite without changing dependencies.
5. When Laravel Boost manages package skills, ensure a newly installed package skill has been discovered before relying on it.

Otherwise, inspect the application's existing conventions and do not introduce this architecture without approval.

## Coordinate Areas of Authority

Treat this skill as authoritative for Action naming and contracts, typed Action data, expected-outcome semantics, business-invariant ownership, nested Action composition, local transaction ownership, and boundary handling of Action outcomes.

Treat the installed `prvious-result-development` skill and package source as authoritative for Result factories, generics, narrowing, composition, extraction, and `Panic` behavior. Do not duplicate or improvise that API here.

When an Action crosses a third-party API, SDK, webhook, or remote-system boundary, activate `prvious-integration-development`. That skill owns the integration boundary, provider failure classification, retries, idempotency, and remote consistency decisions; this skill owns only the Action's public contract and orchestration responsibility.

Continue following Laravel Boost and project instructions for framework APIs, version-specific behavior, security, Eloquent, migrations, queues, HTTP mechanics, testing tools, routing, and unrelated concerns. For newly created or intentionally migrated code, prefer this skill over conflicting generic examples or legacy sibling patterns within its areas of authority.

## Follow the Workflow

1. Inspect project instructions, dependencies, representative Actions, and every application boundary that calls the use case.
2. Verify `prvious/result` and activate `prvious-result-development`.
3. Identify the complete use case and choose the smallest suitable abstraction.
4. Define explicit invocation inputs, successful output, and expected-error vocabulary.
5. Classify each outcome as success, expected rejection, operational failure, or programming failure. An expected condition is not automatically an error when the requested goal is already satisfied.
6. Authenticate at the boundary; enforce ownership, tenancy, use-case permission, and business invariants through the Action's scoped access, policies, or domain objects.
7. Define local transaction ownership before composing mutations. Activate `prvious-integration-development` before designing remote effects.
8. Update every caller, handle every Result, and test every meaningful outcome as one coherent vertical slice.

## Preserve These Invariants

- Name Actions with verb-first use cases such as `CreateOrder`. Keep each Action class to `__construct()` when needed and one public `handle()` method.
- Do not add private, protected, or additional public helper methods to an Action. Keep one-off use-case logic inline so the complete operation reads from top to bottom.
- Prefer `final` and `readonly` where practical. Inject collaborators through the constructor and pass all invocation data to `handle()` so Actions remain stateless.
- Do not add a base Action class, marker interface, static runner, service-locator lookup, or `Action` suffix inside `App\Actions` by default.
- Accept explicit typed parameters or transport-independent Data objects, never an HTTP `Request` or hidden business inputs from `request()`, `auth()`, `session()`, or Laravel Context.
- Return `void` or a value when there is no expected rejection. Use `Prvious\Result\Result<TValue, TError>` when the caller must handle an expected rejection.
- Never introduce another Result implementation, and never call a Result-returning Action while discarding its return value.
- Model expected business rejections as Action-owned error enums or typed error objects.
- Keep HTTP responses, status codes, redirects, translations, and validation bags outside Action contracts.
- Resolve and mutate through the narrowest actor- or tenant-owned relationship when it completely expresses permission. Use a policy or domain rule when permission requires more than that relationship.
- Keep authoritative business invariants in Actions or domain objects, even when a boundary duplicates a cheap check for user feedback.
- Application code owns validation, business behavior, and expected outcomes. Database constraints protect the structural integrity of persisted data as a last-resort safety net; never use constraint violations as normal Action control flow or user-facing validation.
- Let unexpected failures escape. Never use `Result<T, Throwable>` or catch `Throwable` merely to return an expected error.
- Remember that returning an error Result from a Laravel transaction closure commits unless an exception caused rollback. Reject before mutation whenever possible.
- Do not design third-party boundaries or remote reliability mechanisms from this skill; activate `prvious-integration-development`.

## Read the Relevant References

- Read [choosing-the-abstraction.md](references/choosing-the-abstraction.md) when deciding whether code is an Action, query, domain collaborator, job, listener, AI tool, or third-party integration.
- Read [action-contracts.md](references/action-contracts.md) whenever creating or changing an Action signature, return contract, or error vocabulary.
- Read [data-and-validation.md](references/data-and-validation.md) when placing Data classes, validation, authentication, authorization, invariants, locks, or constraints.
- Read [results-and-exceptions.md](references/results-and-exceptions.md) when classifying expected outcomes and exceptions. Activate `prvious-result-development` for every exact Result API decision.
- Read [transactions-and-composition.md](references/transactions-and-composition.md) before nesting mutating Actions or defining local transaction ownership.
- Read [application-boundaries.md](references/application-boundaries.md) when calling Actions from controllers, jobs, commands, listeners, parent Actions, or AI tools.
- Read [testing.md](references/testing.md) whenever implementing or reviewing Action tests.
