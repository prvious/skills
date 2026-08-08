# Integration Failure Classification

Classify a provider outcome by what the application knows, not merely by which SDK exception was thrown.

## Classify Each Outcome

| Provider condition | Application treatment |
| --- | --- |
| Successful terminal response | Application success value |
| Valid non-terminal provider state | Explicit successful status when the use case accepts it |
| Known definitive rejection the caller can handle | Typed expected rejection under the application's adopted contract |
| Temporary provider or network failure | Operational exception |
| Rate limit | Operational exception with retry meaning determined by project policy |
| Connection lost after a remote write may have reached the provider | Unknown-outcome exception or established project state |
| Invalid credentials or provider configuration | Configuration exception |
| Malformed request caused by application code | Programming or integration exception |
| Unknown vendor response or exception | Preserve and propagate, or translate to an application-owned operational exception |

When the application uses the Prvious Laravel Action Pattern, activate `prvious-result-development` before expressing a definitive caller-actionable rejection as Result. Do not teach or guess the package API here.

## Translate Only What Is Known

Catch a vendor exception when the provider documents enough information to classify it or when the application boundary deliberately replaces a vendor exception type with an application-owned operational exception.

Do not catch `Throwable` and turn every failure into the same expected rejection. An outage, invalid credentials, SDK bug, programming error, and definitive business rejection require different behavior.

When translating an operational vendor exception, retain it as the translated exception's `previous` value. This keeps the application contract provider-independent without destroying diagnostic context.

## Protect the Public Contract

Once an application-owned boundary exists, its callers should not need vendor exception classes, numeric provider codes, HTTP status codes, or vendor response objects to decide application behavior. Translate those details inside the provider implementation and expose the smallest truthful application meaning.
