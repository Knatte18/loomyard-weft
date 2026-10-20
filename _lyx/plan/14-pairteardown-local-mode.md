# Card 14 — pairteardown-local-mode

**Edit:**
- `internal/pairteardown#Request`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown/doc.go`
- `internal/pairteardown#TestTeardown_CallSequence`

**Uses:**
- `plan:internal/fabricengine#Topology.RemoveLocal`
- `plan:internal/fabricengine#Topology.SnapshotUncommitted`

**Intent:** D6's teardown seam target: `internal/pairteardown` gains the local mode, which `run-process-ownership` reuses for its own non-owner removal.
`Request` gains `Local` (remove through `Topology.RemoveLocal`: no origin step and no board write), `Snapshot` (D6's force case: keep the task and weft worktrees' uncommitted paths under a local snapshot tag before the removal) and `AfterSnapshot func() error` (run once the snapshot tag is written, which card 18 fills with the lift of the wake-written pair pause entry).
`Teardown` gains a `snapshot` substrate and `New` wires it over `Topology.SnapshotUncommitted`, and wires `remove` to `RemoveLocal` when `req.Local`.
`RemovePair` in local mode: when `req.Snapshot`, it takes the snapshot first (after `EndSession` ended the driver and before the teardown lock is tried), logs its tag and paths at Info and sets them on the returned result's `SnapshotTag` and `SnapshotPaths` (card 13's fields), and runs `AfterSnapshot`; a snapshot refused busy returns that error with nothing removed and `AfterSnapshot` not run, so a re-run still qualifies; an `AfterSnapshot` error is returned with nothing removed, the snapshot tag kept, so a re-run takes the earlier-snapshot path.
It then removes the pair and skips `settleBoardClaim`, so the local mode writes nothing to the board.
A busy teardown or weft write lock from `RemoveLocal` surfaces as `fabricengine.ErrPairBusy`.
The quiet rule and every refusal stay: a local-mode caller passes `RefuseWhenBusy` with a zero `QuietWait`, so a busy driver is `ErrDriverBusy` at once, except the force case, whose caller passes no `RefuseWhenBusy` and `Force`, as batten's own teardown does; the two `EndSession` waits, the loop lock's seizure and the run lock's release, stay, each bounded by `runLockWait`.
`doc.go` gains a Local mode section stating this and its bound.

`TestTeardown_CallSequence` gains rows over its fakes: local mode calls the local remove and never the board claim reads or writes; local with snapshot calls snapshot, then `AfterSnapshot`, then remove, in that order; a snapshot returning `ErrPairBusy` stops before `AfterSnapshot` and the remove; local mode with a busy driver and `RefuseWhenBusy` returns `ErrDriverBusy` with nothing ended; an `AfterSnapshot` error stops before the remove; a local remove returning `fabricengine.ErrPairBusy` surfaces it from `RemovePair` with the board untouched; the force case with `Force` over a refusal probe answering a merge in progress returns that refusal with nothing ended and no snapshot taken.
`New`'s wiring of `remove` to `Topology.RemoveLocal` and of `snapshot` to `Topology.SnapshotUncommitted` is reached over a real hub by card 20's force-case step of the cleanup verb test, which removes an ended pair through `pairteardown.New`'s local mode; pairteardown's own tagged tests cannot build a hub, since the package sits in `fabriccli`'s dependency set and may not import `hubforge`, and its `TestRun_TeardownScenario` is in the `tmux` tier no verify runs.

**ImpactSummary:** Additive fields on `Request` and `Teardown`; the two existing callers (`lyx fabric remove`, batten's teardown) leave `Local` false and behave as today.

**Commit:** 14: pairteardown: a local mode with no origin step and no board write
