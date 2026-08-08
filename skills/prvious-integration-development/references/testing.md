# Testing Integrations

Test the application boundary and the provider translation separately. Use project conventions and provider-supported fakes, SDK test utilities, or controlled HTTP responses.

## Test Application Consumers

When an Action depends on an application-owned integration boundary, use a fake or controlled implementation of that boundary in the Action test. Assert the Action's success, expected rejection, and exception behavior without coupling the test to vendor classes.

When the application intentionally uses a concrete provider adapter without an interface, use the smallest supported fake or HTTP transport seam rather than inventing an interface solely for mocking.

## Test Provider Translation

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

Do not duplicate `prvious/result` package tests. When Result is part of an integration contract, activate `prvious-result-development` and test only the application's mapping and durable behavior.
