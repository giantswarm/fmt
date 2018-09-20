# Idempotency Conventions

Infrastructure provisioning code should more often than not be idempotent.
Therefore we have this naming convention established so we can expect certain
behaviors from our APIs.

## Imperative API

- Name of the function should be imperative verb like `Create`, `Update` or
  `Delete`.
- Some expected common errors matched by `IsNotFound` or `IsAlreadyExists` may
  occur during execution.

## Idempotent API

- Name of the function should be a past tense verb prefixed with `Ensured`,
  e.g. `EnsureCreated` or `EnsureDeleted` (note `EnsureUpdated` makes little
  sense in idempotent APIs).
- Such APIs should return errors only in exceptional circumstances like
  depending API being down, or reached request limit.
