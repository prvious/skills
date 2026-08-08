# Choosing the Abstraction

Classify a responsibility before extracting it. Treat an Action as a complete application use case, not a renamed helper method or a container for unrelated behavior.

## Choose by Responsibility

| Responsibility | Use |
| --- | --- |
| Complete application use case | Action |
| Reusable complex read | Query object |
| Reusable domain calculation | Value object or domain service |
| Asynchronous delivery, scheduling, and retry | Job calling an Action |
| HTTP translation | Controller, Form Request, or Resource |
| Event reaction | Listener calling an Action |
| Model-selectable bounded capability | AI tool calling an Action |
| Third-party API, SDK, webhook, or remote system | Activate `prvious-integration-development` |

Use an Action for an operation that expresses an application goal, coordinates domain behavior, and has a meaningful caller-visible contract.

Good Action names include:

```text
CreateOrder
MergeEntries
InviteTeamMember
CancelSubscription
RecalculateLoyaltyPoints
```

Keep these as smaller collaborators rather than Actions:

```text
FormatCurrency
FindUserById
MapOrderToArray
NormalizePhoneNumber
```

## Avoid Ambiguous Services

Do not create `OrderService`, `OrderManager`, or `OrderProcessor` as a container for unrelated use cases. Split complete operations into Actions. Retain a domain service only for reusable domain behavior that does not naturally belong to an entity or value object. Do not create a collaborator solely to remove one-off logic from an Action; when no independent responsibility exists, keep that logic inline in `handle()`.

Do not create an interface for every Action or collaborator. Add one when it represents a meaningful application port or another deliberate project boundary. Defer decisions about third-party gateways, adapters, provider clients, and SDK wrappers to `prvious-integration-development`.

## Preserve Delivery Boundaries

- Let a Job provide queue delivery and retry behavior; place the use case in an Action.
- Let a Listener translate an event into an Action call; do not duplicate the mutation.
- Let a Controller translate HTTP input and output; do not move HTTP types into the Action.
- Let an AI tool expose a bounded model-callable capability; validate tool input and delegate deterministic behavior to an Action.

One feature may use several abstractions. Choose each boundary by responsibility instead of forcing the entire feature into one class type.
