# Plan-Review round 2 — fixer report

All three findings were checked against the code and fixed.
`lyx loom validate-plan` passes after the edits, with only the two informational `create-new-unit` notes.

## Fixed

- **F1** (`02-shared-hub-board.md`): dropped the locked-entry row from `TestSetRunStatus`.
  It keeps the no-status, `done` and absent rows, and a sentence says why the status-only pass gets no row: the lock judges every write's net change in `boardCriticalSection`, whichever `Board` method makes it, and Card 1's `TestRunLock` `SetStatus` rows already pin that.
  The `boardengine` budget raise stays at one, because the test itself is still new.
- **F2** (`03-batten-claim.md`): confirmed the premise.
  `CreateWorktree` loads the fabric config from `fabricengine.BoardDir(location.HubPath)` (`internal/battencli/wire.go:587`).
  A bare `t.TempDir()` hub fails in `FindBaseDir` with `ErrNotInitialized`, which is not the `config file … not found` error the first row gets (`internal/configengine/config.go`).
  The second row's fixture is now a hub whose board directory holds only an empty `configengine.ConfigDir`.
  On that hub `OpenHub` fails on the missing board config, and the closure's error names the fabric config file and not the board's, which shows the run went past the failed claim.
  The row also asserts, through `logcapture.Capture`, a `logger.Warn` that names the slug.
  "The same fabric-config error" and "writes nothing" are removed.
  The no-`t.Parallel` sentence now names the test and adds the logger redirect to the process-global state it touches, since `logcapture.Capture` sets the logger's global output.
- **F3** (`02-shared-hub-board.md`): `markDoneFixtureDir` now also gets its board handle from `boardengine.OpenHub(location.HubPath)` instead of its own copy of the opener's body (`internal/loomcli/landingdeps_markdone_test.go:41-46`).

## Disputed

None.

## Deferred

None.
