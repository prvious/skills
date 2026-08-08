# Application Boundaries

Keep Actions transport-independent. Let each caller translate its input and the Action's outcome into boundary-specific behavior. Never call a Result-returning Action while discarding its return value.

## Controllers

Make a controller:

1. Receive validated input.
2. Obtain the authenticated actor and relevant tenant context.
3. Construct typed Action input.
4. Invoke one top-level Action with every business-relevant input explicitly.
5. Handle every Action outcome and map it to HTTP behavior.

```php
$result = $createOrder->handle(
    actor: $request->user(),
    data: CreateOrderData::from($request->validated()),
);

// Handle both Result variants using prvious-result-development.
```

A Form Request, gate, or controller may reject an unauthorized HTTP request early. The Action must still enforce reusable ownership, tenancy, permission, and business rules through scoped access, policies, or domain objects.

Keep status codes, redirects, translated messages, validation bags, and Resources at the HTTP boundary. Judge controller thinness by responsibility rather than a literal line limit; exhaustive outcome-to-HTTP mapping is boundary work.

## Jobs

Treat a returned expected error and a thrown exception differently:

```text
Returned error Result  normal PHP return; Laravel considers the job successful
Thrown exception       queue failure with Laravel retry/failure behavior
```

Handle every expected error explicitly. Ignore, record, release, reschedule, or fail intentionally only according to its business meaning. Let unexpected and retryable exceptions escape. Do not catch `Throwable`, log it, and return normally.

Add `failed()` only when exhausted failure requires cleanup, state repair, compensation, or notification. Follow Laravel Boost and project queue conventions for delivery, retry, uniqueness, and other queue mechanics.

## Commands and Listeners

Convert expected errors into exit codes, console output, recorded state, or event-specific behavior. Allow unexpected failures to remain exceptions. Call the Action rather than copying its authorization, invariants, or mutations.

## Parent Actions

Handle or propagate every child Result deliberately. Translate child error vocabularies into the parent's contract and inspect local mutation and transaction ownership before composing outcomes. Do not expose a child's implementation-specific errors unless they deliberately belong to the parent use case.

## AI Tools

Expose only bounded, model-selectable capabilities. Validate and authenticate tool input, construct explicit Action input, call the Action, and map every outcome into a predictable tool result. The Action remains responsible for authoritative permission and business invariants. Keep deterministic required orchestration outside the model and never grant an AI tool permission merely because the underlying Action can perform the operation.
