# Action Contracts

Define an Action around one use case with explicit, transport-independent inputs and the smallest truthful return type.

## Use the Standard Shape

```php
<?php

declare(strict_types=1);

namespace App\Actions\Orders;

use App\Data\Orders\CreateOrderData;
use App\Data\Orders\CreatedOrderData;
use App\Models\User;
use Prvious\Result\Result;

final readonly class CreateOrder
{
    /**
     * @return Result<CreatedOrderData, CreateOrderError>
     */
    public function handle(User $actor, CreateOrderData $data): Result
    {
        // Implement the use case.
    }
}
```

Use verb-first names. Prefer `CreateOrder` over `CreateOrderAction` inside `App\Actions`. An Action class defines only `__construct()` when it has injected collaborators and one public `handle()` method. Do not add private, protected, or additional public helper methods.

Actions must be stateless between calls:

- Constructor arguments are long-lived collaborators.
- `handle()` arguments are invocation-specific data such as actor, tenant, target identifiers, command Data, or business date.
- Do not store invocation data in mutable properties.
- Do not read invocation data through `request()`, `auth()`, `session()`, Laravel Context, or another ambient source.

Use constructor injection for collaborators. Prefer `final` and `readonly` where practical. Do not introduce a base Action, marker interface, static runner, or `app()`/`resolve()` lookup.

## Keep the Complete Use Case in View

Write `handle()` as a top-to-bottom account of the complete use case. Do not extract private or protected methods merely to reduce its line count; a somewhat longer linear method is preferable to making readers jump around the class to discover the operation.

When logic has another truthful owner, move it there:

- Extract another Action only when the operation is an independently meaningful application use case with its own contract.
- Put reusable domain behavior or calculations on the appropriate entity, value object, policy, or domain service.
- Put reusable complex reads in an established model scope, relationship, or query object.
- Keep input validation and presentation mapping at the relevant application boundary.

If one-off logic has no independently meaningful abstraction, leave it inline. Do not invent a child Action, generic service, trait, utility class, or immediately invoked closure solely to make `handle()` look shorter. Closures structurally required by an API, such as a transaction closure, remain inline at the point where their behavior matters.

## Choose the Return Contract

| Situation | Contract |
| --- | --- |
| No meaningful output and no expected rejection | `void` |
| Successful value and no expected rejection | `TValue` |
| Caller must handle expected rejection | `Result<TValue, TError>` |
| Expected rejection and no success payload | `Result<null, TError>` |
| Unexpected operational failure | Exception |
| Programmer misuse or internal-contract violation | Exception |

The Prvious Laravel Action Pattern requires `prvious/result`, but not every Action returns Result. Do not encode meaningful rejections as `bool`, `false`, `null`, status strings, or raw arrays. Declare both Result generic types in PHPDoc and follow the activated `prvious-result-development` skill for exact package usage.

Returning an Eloquent model is valid when it is the natural success contract. Create output Data only when callers benefit from a stable, transformed, serialized, or transport-independent value.

Never ignore a Result-returning Action call. A caller must handle the outcome, translate it, compose it, or return it to a caller that deliberately owns handling it.

## Own the Error Vocabulary

Use an enum when errors need identity only:

```php
enum CreateOrderError
{
    case AlreadySubmitted;
    case InsufficientInventory;
    case InvalidState;
}
```

Use typed objects behind an Action-specific interface when errors carry context:

```php
interface TransferFundsError {}

final readonly class InsufficientBalance implements TransferFundsError
{
    public function __construct(
        public int $requiredCents,
        public int $availableCents,
    ) {}
}
```

The Action owns its public expected-error vocabulary. Translate child or domain errors unless they deliberately belong to the parent use case. Avoid a global `ApplicationError`, magic strings, associative error arrays, `Throwable` error types, translated messages, HTTP status codes, redirects, and response objects.
