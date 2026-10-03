# Card 28 — statuscommit-seam

**Create:**
- `internal/statuscommit/statuscommit.go`
- `internal/statuscommit/statuscommit_test.go`

**Edit:**
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_commitstatus_test.go`
- `internal/battencli/commitstatus.go`
- `internal/battencli/commitstatus_test.go`
- `docs/overview.md`

**Uses:**
- `internal/loomcli/wiring_commitstatus_integration_test.go`

**Intent:** Move the commit-status seam duplicated in `loomcli` and `battencli` into one production package below both CLIs, `internal/statuscommit`, and test it once there (D15).
It owns the deps struct (`MergeActive`, `Commit`, `Push`), the message renderer and the three dispositions in their current order: skip while mid-merge (a probe error counts as active), commit hard-errors except when the failure re-probe finds a merge gone live (or the re-probe errors), push warns on every error.
The per-module texts stay byte-identical by parameter: each caller tells the seam its commit-message prefix (`loom`, `batten`) and its log prefix (`loomcli`, `battencli`), so every commit message and log line reads as it does today.
`statuscommit` imports the standard library and `internal/logger` only; it derives no path.
`loomcli` keeps its board-status write ahead of the core and its `loomCommitStatusDeps`/`statusCommitPathspec`;
it also keeps its own `commitStatusDeps` type with its flat field set (`SetBoardStatus` included) and its `newCommitStatusSeam(deps)` signature, now a wrapper that does the board-status write and calls the shared core.
`commitrecords.go`'s `commitRecordsVerb`, `commitrecords_test.go`'s composite literals and the protected `wiring_commitstatus_integration_test.go` use both, and none of them changes in this card;
only the duplicated message renderer, failure disposition and core closure move.
`battencli` keeps its on-disk no-op-transition marker skip ahead of the core and its marker write between a successful commit and the push, plus `battencli`'s `battenCommitStatusDeps`/`battenRunCommitPaths`.
The core takes an optional after-commit callback, run after a successful commit and before the push;
`battencli` passes its marker write there and `loomcli` passes none.
The duplicated disposition tests (`CommitErrorPropagates`, `CommitFailsAfterMergeWentLive_TakesTheSkip`, `PushErrorReturnsNil`, `PushRejectedReturnsNil`, `MergeActiveSkips`, `CommitStatusMessage`, and `loomcli`'s `CommitFailsAndReProbeFails_TakesTheSkip`) become one set in `statuscommit_test.go`;
`battencli` keeps only its marker-skip tests and `TestBattenRunCommitPaths`, `loomcli` its board-status, wiring and pathspec tests.
What moving `CommitStatusMessage` lets through: the shared test renders whatever prefix it passes, so a caller passing the wrong prefix would go unnoticed there.
The bound: `loomcli`'s `loom:` subject stays pinned by `wiring_commitstatus_integration_test.go`, and one of `battencli`'s kept marker-skip tests asserts the message its fake `Commit` receives is `batten: <producer> -> <state>`, so each caller's commit prefix stays pinned in the layer it keeps.
The log prefixes become caller-supplied and stay unpinned by any test, as they are today.
Behaviour and log text are unchanged;
`wiring_commitstatus_integration_test.go` is protected (D21) and must pass untouched.
`docs/overview.md`'s package tree gains `internal/statuscommit/`.
Deleted and moved tests go in the commit body.

**ImpactSummary:** Two CLI packages lose their private copies of the disposition core and call one new seam package; `loomcli`'s deps type and seam constructor keep their shape, so `commit-records` and the protected integration test are untouched, and no behaviour or operator-facing text changes.
