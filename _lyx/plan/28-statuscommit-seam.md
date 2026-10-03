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
`battencli` keeps its on-disk no-op-transition marker skip ahead of the core and its marker write between a successful commit and the push, plus `battencli`'s `battenCommitStatusDeps`/`battenRunCommitPaths`.
The duplicated disposition tests (`CommitErrorPropagates`, `CommitFailsAfterMergeWentLive_TakesTheSkip`, `PushErrorReturnsNil`, `PushRejectedReturnsNil`, `MergeActiveSkips`, `CommitStatusMessage`, and `loomcli`'s `CommitFailsAndReProbeFails_TakesTheSkip`) become one set in `statuscommit_test.go`;
`battencli` keeps only its marker-skip tests and `TestBattenRunCommitPaths`, `loomcli` its board-status, wiring and pathspec tests.
Behaviour and log text are unchanged;
`wiring_commitstatus_integration_test.go` is protected (D21) and must pass untouched.
`docs/overview.md`'s package tree gains `internal/statuscommit/`.
Deleted and moved tests go in the commit body.

**ImpactSummary:** Two CLI packages lose their private copies and call one new seam package; no behaviour or operator-facing text changes.
