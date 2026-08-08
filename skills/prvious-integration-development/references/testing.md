# Testing Integrations

Test application behavior and provider translation as separate layers. A consumer test should know the application-owned contract but nothing about the vendor. An adapter test should prove how vendor behavior becomes that contract without contacting the real provider.

## Use the Correct Test Double

| Subject under test | Test double | What the test proves |
| --- | --- | --- |
| Action or other consumer of an application-owned interface | Purpose-built Fake implementing that interface | Application behavior for each application-level outcome |
| Concrete provider adapter | Provider-supported SDK fake or controlled HTTP transport | Request mapping, response translation, and failure classification |
| Optional provider smoke test | Provider sandbox or test account | Selected real connectivity assumptions outside the ordinary suite |

Do not create an application-owned interface solely to make testing possible. When a concrete adapter is the smallest truthful boundary, test it through the provider SDK's supported fake or its HTTP transport seam.

## Provide a First-Class Fake

Whenever the application owns an integration interface, add a deterministic Fake in the project's test-support namespace, such as `Tests\Fakes`. Use it instead of Mockery or another dynamic mock in tests of Actions and other consumers.

A good Fake:

- Implements the application-owned interface.
- Accepts and records only application-owned inputs.
- Returns explicitly configured application-owned outcomes or throws explicitly configured exceptions.
- Fails loudly when a call has no configured outcome.
- Starts fresh for every test and has no shared static state.
- Never contacts the provider or contains credentials.
- Does not reproduce vendor payloads, status codes, SDK types, timing, or internal provider behavior.
- Does not duplicate the real adapter's translation logic.

The Fake is a controllable implementation of the application contract, not a simulator for the provider. Adapt this shape to the boundary's actual types:

```php
<?php

namespace App\Integrations\Workspaces;

interface WorkspaceProvisioner
{
    public function provision(WorkspaceProvisioningRequest $request): ProvisionedWorkspace;
}
```

```php
<?php

namespace Tests\Fakes;

use App\Integrations\Workspaces\ProvisionedWorkspace;
use App\Integrations\Workspaces\WorkspaceProvisioningRequest;
use App\Integrations\Workspaces\WorkspaceProvisioner;
use LogicException;
use Throwable;

final class FakeWorkspaceProvisioner implements WorkspaceProvisioner
{
    /** @var list<WorkspaceProvisioningRequest> */
    private array $requests = [];

    /** @var list<ProvisionedWorkspace|Throwable> */
    private array $outcomes = [];

    public function willReturn(ProvisionedWorkspace $workspace): self
    {
        $this->outcomes[] = $workspace;

        return $this;
    }

    public function willThrow(Throwable $exception): self
    {
        $this->outcomes[] = $exception;

        return $this;
    }

    public function provision(WorkspaceProvisioningRequest $request): ProvisionedWorkspace
    {
        $this->requests[] = $request;

        if ($this->outcomes === []) {
            throw new LogicException('No FakeWorkspaceProvisioner outcome was configured.');
        }

        $outcome = array_shift($this->outcomes);

        if ($outcome instanceof Throwable) {
            throw $outcome;
        }

        return $outcome;
    }

    /** @return list<WorkspaceProvisioningRequest> */
    public function requests(): array
    {
        return $this->requests;
    }
}
```

This example's declared contract has only a successful return value. If a real boundary returns a typed outcome or Result, make the Fake queue that complete return type. Reserve `willThrow()` for exceptions; never turn a definitive expected rejection into an exception merely to fit the Fake.

Use the Fake through the same interface the application consumes:

```php
$fake = (new FakeWorkspaceProvisioner())
    ->willReturn($provisionedWorkspace);

$this->app->instance(WorkspaceProvisioner::class, $fake);

$actual = $this->app
    ->make(CreateWorkspace::class)
    ->handle($command);

$this->assertEquals([$expectedRequest], $fake->requests());
$this->assertSame($provisionedWorkspace, $actual);
```

Assert meaningful application inputs, returned outcomes, durable state, and propagated exceptions. Do not recreate mock-style call-order expectations unless order is part of the application contract. Configure every meaningful return variant and operational failure consumers must handle. When the contract uses `Prvious\Result`, activate `prvious-result-development` rather than inventing Result helpers in the Fake.

## Test the Provider Adapter Separately

Provider-adapter tests should use controlled provider responses and inspect only contract-relevant outbound details. For an adapter built on Laravel's HTTP client, the shape is:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::fake([
    'https://api.vendor.example/workspaces' => Http::response([
        'id' => 'workspace_123',
    ], 201),
]);

$workspace = $this->app
    ->make(VendorWorkspaceProvisioner::class)
    ->provision($request);

$this->assertSame('workspace_123', $workspace->providerId);

Http::assertSent(
    fn (Request $request): bool =>
        $request->method() === 'POST'
        && $request->url() === 'https://api.vendor.example/workspaces',
);
```

Use an official SDK fake when one exists and exercises the adapter's actual seam. Do not wrap an SDK in another interface merely because its test utility uses a name such as fake, stub, mock handler, or test transport; the relevant requirement is deterministic behavior with no real remote request.

Cover every applicable provider behavior:

- A successful response becomes the correct application value.
- Each supported definitive rejection becomes the correct typed application outcome.
- Provider outages and temporary network failures remain operational exceptions.
- Invalid credentials and malformed application requests are not disguised as expected business rejection.
- A possible-write timeout is not treated as a definitive failure.
- Translated operational exceptions preserve the original vendor exception as `previous`.
- Unexpected response shapes or undocumented exceptions do not silently become a generic expected error.
- No real remote requests escape the test suite.

Test retries, idempotency identities, pending states, reconciliation, compensation, or durable handoff only when the application has explicitly adopted those mechanisms. Test webhook authentication and the provider's documented duplicate or ordering behavior when webhooks are in scope.

## Keep Live Provider Tests Separate

Do not make ordinary unit or feature tests depend on credentials, network availability, provider state, or a shared sandbox. If the project deliberately maintains provider smoke or contract tests, keep them opt-in, narrowly scoped, and separate from the deterministic suite. They supplement Fake and adapter tests; they do not replace them.

Do not duplicate `prvious/result` package tests. When Result is part of an integration contract, activate `prvious-result-development` and test only the application's mapping and durable behavior.
