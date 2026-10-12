## Changes

- **F1** (fixed): `attempt.json` now goes through `internal/state`.
  - `WriteAttempt` uses `state.WriteJSON` under `<attempt dir>/attempt.lock`, so a rewrite keeps keys `Attempt` does not declare.
  - `readAttemptFile` uses `state.ReadJSONNoMkdir`.
  - `Close` and `MarkApplied` use a new `updateAttempt` helper over `state.UpdateJSON`, so the read-modify-write holds the write lock.
    `Close` still reads first to skip an already-closed attempt and to stop the strand before marking closed.
  - `Begin` and `failBegin` keep calling `WriteAttempt`, which is now `state.WriteJSON`.
  - `internal/state` joined the import allowlist in `seam_enforcement_test.go`; it takes paths from its caller, so the Told-Geometry Invariant holds.
  - The `MarkApplied` subtest in `session_test.go` now injects an unknown top-level key into `attempt.json` and asserts it survives the rewrite.

`lyx gate test ./internal/alignment` and `./internal/loomcli` pass.

## Disputed

None.

## Deferred

None.
