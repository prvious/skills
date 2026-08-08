# Choosing the Integration Boundary

Choose an abstraction because of responsibility, not because a vendor namespace appears in the code.

## Choose the Smallest Useful Shape

| Responsibility | Use |
| --- | --- |
| Stable local computation with no remote behavior | Use the maintained library directly |
| Provider-specific translation contained in one place | Concrete provider adapter or client |
| Meaningful application capability that must be independent of a provider | Application-owned interface with provider adapter |
| Container bindings and provider configuration | Laravel service provider |
| Complete application goal that uses the integration | Action |
| Queue delivery, scheduling, and retry mechanics | Job calling an Action or integration boundary |
| Receiving and authenticating provider callbacks | Webhook controller or handler delegating the state transition |

A concrete class is enough when the application needs one implementation and can test it through the provider's supported fake or HTTP test facilities. Add an interface when it represents a stable application capability, supports meaningful substitution, isolates provider-specific contracts, or is already an established project port.

Do not create an interface for every SDK class. Do not make an application-owned interface reproduce the vendor API method for method; define it around the capability the application actually needs.

## Keep Responsibilities Separate

An Action coordinates the use case. A provider adapter translates between application meaning and vendor behavior. A Laravel service provider binds implementations and configures dependencies; it is not a container for business logic.

Avoid broad classes such as `IntegrationService`, `ApiManager`, or `ProviderService` that collect unrelated capabilities. Prefer a specific capability or provider responsibility, such as `ChargePayments`, `ProvisionWorkspace`, or `StripePaymentGateway`, according to the boundary being represented.

Do not leak vendor DTOs or exceptions into an application-owned boundary merely to avoid writing a small translation. Conversely, do not create duplicate application DTOs when the vendor type is intentionally confined to provider-specific code and no stable application contract needs it.

## Route Application Behavior

When the integration participates in a complete use case, let the Action depend on the selected boundary and keep HTTP, queue, console, and webhook delivery concerns outside it. Activate `prvious-action-development` when changing that Action contract.
