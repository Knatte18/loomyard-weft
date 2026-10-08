# Webster round 3 — fixer report

Both findings were valid against card 15 and the code, and both are fixed, each in its own commit.
`go test ./...` passes, and `go test -tags integration` passes for `internal/websterengine`, `internal/webstercli` and `cmd/...`.
`lyx loom lint-comments` is clean.

## Fixed

### F1 — standalone resets to targets other than start listed no uncommitted paths

Commit `13db5e957`.
`resetCmd` now reads `UncommittedPaths` after the move for every standalone reset as well as for every reset to start, carrying `uncommitted` and `warnings` (a failed listing becomes a warning, as on the start path); `moved` and `reason` stay start-only.
The long help and the reset section of `internal/websterengine/doc.go` now name the key for every standalone reset.
This was a coverage gap, so `TestResetCmd`'s standalone step became "standalone resets move HEAD with git's keep form and list what they left": it first runs `--to pre-fix` over an extra commit, asserting HEAD and `uncommitted`, then runs the existing `--to start` assertions.
The new pre-fix assertion fails against the previous `reset.go` (`pre-fix uncommitted = <nil>`).

### F2 — the recover-batch uncommitted-path refusal had an unnamed and untested second cause

Commit `aa882a6a0`.
The `recover-batch: uncommitted paths unreadable` row in `contracts/specs/refusal-spec.md` now also names a failed audit of the run's sessions, or a failure to resolve their written paths, as a trigger.
`TestRecoverSpawnOrAttach_StartFailures` gains a row with a dirty path, a recorded `MasterSessionID` and an erroring `AuditForks`, reusing the test's assertions: the error wraps the audit error and ends with the `recover-batch 1` transient re-run, no batch state, no prepared strand, and the retry spawns.
The test's doc comment names the new failure.

## Disputed

None.

## Deferred

None.
