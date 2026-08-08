# Data, Validation, and Authorization

Keep Action contracts reusable across HTTP, queues, commands, listeners, tests, and other Actions.

## Organize Data by Feature

Prefer feature-first namespaces:

```text
app/
├── Actions/
│   └── Entries/
│       ├── MergeEntries.php
│       └── MergeEntriesError.php
└── Data/
    └── Entries/
        ├── MergeEntriesData.php
        └── MergedEntryData.php
```

Keep actual HTTP Form Requests in `App\Http\Requests` and Resources in `App\Http\Resources`. Avoid broad warehouses such as `App\Data\Requests`, `App\Data\Actions`, or `App\Data\Responses`.

Use Spatie's `laravel-data` package when its construction, validation, transformation, serialization, or TypeScript features provide value. Use a plain readonly DTO when they do not. Do not create a Data class merely to wrap one already well-typed parameter.

## Assign Handling Deliberately

| Concern                                                                             | Primary handling                                                                                             |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Required field, scalar type, UUID format, or string length at an untrusted boundary | Form Request or input boundary                                                                               |
| User-facing uniqueness or referenced-record feedback                                | Form Request or input boundary before calling the Action                                                     |
| Authenticating the caller and obtaining the actor                                   | Application boundary                                                                                         |
| Actor, tenant, or account ownership of the target                                   | Action or domain object through a scoped relationship, query, or policy                                      |
| Use-case permission not fully expressed by ownership                                | Action or domain object, often through a policy                                                              |
| Business meaning of an existing, duplicate, missing, or inaccessible record         | Action or domain object through a typed expected outcome                                                     |
| State-transition validity                                                           | Action or domain object                                                                                      |
| Sufficient balance or inventory                                                     | Action or domain object                                                                                      |
| Race-sensitive state                                                                | Action or domain object at the mutation point, using an established lock or atomic persistence operation    |
| Last-resort relational integrity under concurrent writes                            | Ordinary unique, foreign-key, and nullability schema constraints as a safety net, not Action control flow    |

Pass every business-relevant input explicitly, including actor, tenant, account, and business date. Use Laravel Context for diagnostics such as trace and correlation IDs, not hidden authorization or mutation inputs.

## Validate Early Without Using Database Exceptions as Control Flow

Use Laravel's `unique` and `exists` validation rules at each untrusted input boundary when they provide useful feedback before the Action runs. Do not wait for a database exception to produce a normal validation response.

Boundary validation is not the authoritative protection for a reusable use case. Jobs, commands, listeners, and parent Actions may call the Action later or through another path, and concurrent requests may both pass the same preflight check. The Action or domain model must still express the business meaning of an existing, duplicate, missing, or inaccessible record when callers are expected to handle it.

Keep ordinary unique, foreign-key, and nullability constraints as a final storage safety net against races and programming mistakes. Their presence does not make a raw database exception part of the Action contract and does not justify introducing database enums, triggers, stored procedures, or custom database logic.

Do not catch broad `QueryException` instances inside Actions to implement normal business branching, and do not write Action tests that expect a raw database exception for an ordinary duplicate or missing reference. Test the boundary validation and the Action's typed outcome instead. When a concurrent conflict must remain an expected outcome, use an established atomic Laravel or project persistence operation rather than scattering ad hoc exception translation across Actions.

## Prefer Relationship-Scoped Access

Resolve and mutate a target through the narrowest relationship that truthfully represents ownership or tenancy:

```php
$course = $actor->ownedCourses()->find($courseId);

if ($course === null) {
    // Return the Action's typed not-found or inaccessible rejection.
}

$course->update($attributes);
```

When `ownedCourses()` completely expresses who may perform the operation, the scoped lookup is the authoritative authorization check and a duplicate policy call is unnecessary. Prefer this structural enforcement over loading an unrestricted model and comparing foreign keys afterward.

Do not treat an arbitrary association as permission. A relationship named `courses()` may describe enrollment rather than ownership, and permission may still depend on a role, state, capability, or allowed field. Use an explicit policy or domain rule when the relationship alone does not express the complete rule.

Always constrain the intended target before a mutation. Calling `update()` or `delete()` on an unconstrained to-many relationship affects every record matched by that relationship and may bypass per-model behavior expected by the application.

## Enforce Invariants for Every Caller

A Form Request or controller may perform an early authorization or duplicate a cheap business check for HTTP behavior and field-level feedback. Never make that boundary check the sole protection for a reusable mutation because jobs, commands, listeners, and parent Actions bypass it.

Enforce authoritative ownership, tenancy, permission, and business invariants through the Action's scoped access, policy, or domain objects. Protect race-sensitive checks at the mutation point with an established lock or atomic persistence operation; validation performed earlier cannot guarantee mutable state remains unchanged. Keep schema constraints as the final integrity safeguard, never the application's normal decision mechanism.
