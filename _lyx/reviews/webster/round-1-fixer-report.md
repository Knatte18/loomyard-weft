## Changes

- F1 (`internal/orchengine/cleanup.go`): `cleanupHandoffs` skips the state lock acquire in a dry run when `p.StateLockPath` is absent; apply and a dry run with an existing lock file still take the blocking lock.
  `TestCleanupHandoffs` gains the row "an absent state lock file is not created by a dry run", which deletes the lock file first and asserts it is still absent after the dry run.
- F2 (`internal/battencli/cleanup.go`): the `CleanupReport.Applied` comment now says it reports whether the pass ran with apply true.

`lyx gate test ./internal/orchengine ./internal/battencli` is green.

## Disputed

None.

## Deferred

None.
