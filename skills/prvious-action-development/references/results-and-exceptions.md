# Results and Exceptions

This reference owns Action-level outcome semantics. Activate the installed `prvious-result-development` skill and inspect the installed package for every exact Result API, generic, narrowing, composition, extraction, or `Panic` decision.

## Classify Each Outcome

| Condition | Treatment |
| --- | --- |
| The requested goal was fulfilled | Success value, including an appropriate idempotent no-op |
| The requested goal was not fulfilled and the caller must handle a normal alternative | Typed Action-owned error Result |
| Database, filesystem, framework, or other operational failure | Exception |
| Third-party provider or remote-system outcome | Activate `prvious-integration-development` |
| Invalid configuration | Exception |
| Programmer misuse or violated internal contract | Exception |

An expected condition is not automatically an error. An error Result means the Action did not fulfill its promised goal and the caller must choose alternate behavior.

For example, `AlreadySubmitted` may be a rejection from `SubmitOrder` when the requested state transition was invalid. The same existing state may be success from `EnsureOrderSubmitted` because that Action's goal is already satisfied. Classify duplicate, missing, or already-processed state according to the use-case contract rather than a universal list.

Expected rejections may include insufficient balance or inventory, invalid state transitions, inaccessible records, or another foreseeable condition the caller can deliberately handle. Let unexpected operational and programming failures remain exceptions.

Do not catch an exception merely to return `false`, `null`, or an expected error containing the exception. Do not use `Result<T, Throwable>` as a generic safety net. Catch only failures that can be deliberately classified, and preserve the original exception as `previous` when translating one operational exception into another.

## Require Deliberate Result Ownership

Never discard the return value of a Result-returning Action. Each caller must do one of the following:

- Handle both variants and translate them into boundary-specific behavior.
- Translate the child Action's expected-error vocabulary into its own public vocabulary.
- Compose or propagate the Result to a caller that deliberately owns handling it.

Do not extract a success value merely to avoid dealing with an expected rejection. Follow `prvious-result-development` for safe matching, narrowing, transformation, composition, and extraction.

## Translate Child Errors

A parent Action owns its public expected-error vocabulary. Translate implementation-specific child errors unless they deliberately mean the same thing in the parent use case. Do not expose a union of unrelated child errors merely because composition made that convenient.

Before composing mutating child Actions, inspect local transaction ownership and rejected-path writes. Result composition must not hide partial mutations, commit an earlier write after a child rejection, or make retry and side-effect behavior unclear.
