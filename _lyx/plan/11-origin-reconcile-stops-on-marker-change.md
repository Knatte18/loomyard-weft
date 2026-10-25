# Card 11 — The origin reconcile leaves a weft unmoved across a marker change

**Edit:**
- `internal/fabricengine#originMover.move`
- `internal/fabricengine#BranchState.stops`
- `internal/fabricengine#BranchOutcome`
- `internal/fabricengine#BranchOutcome.stopWayForward`
- `internal/fabriccli#stopsRemoval`
- `internal/hibernatecli#stopsEntries`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/cleanupcli#sharedStopText`
- `internal/adoptcli#sharedStopText`
- `internal/adoptcli#stoppedBranches`
- `internal/adoptcli/doc.go`
- `internal/adoptcli#TestAdopt`
- `internal/battencli#judgeOrigin`
- `internal/battencli#TestCancelRun`

**Create:**
- `plan:internal/fabricengine#BranchAnchorStopped` -> `const BranchAnchorStopped BranchState = "anchor_stopped"`

**Uses:**
- `plan:internal/fabricengine#weftAnchorStop`
- `plan:internal/fabricengine#refuseEmptyOwnMarker`
- `internal/lyxcwd#NamedWorktree`
- `internal/fabricengine#reconcileRecordsBranch`
- `internal/fabricengine#ReconcileOrigin`

**Intent:**
D10's origin reconcile paragraph.
`originMover.move`, for a records branch that is not Bolt and needs a move (fast-forward or replay), hub-wide or run-scoped, first refuses an empty own marker of that weft and then runs `weftAnchorStop` on its tip and origin's head, with the pair's (or prime's) `NamedWorktree` anchor as fallback.
Either stop leaves the weft unmoved and records `BranchAnchorStopped` on its outcome, with `BranchOutcome` gaining `Anchor string`, the anchor the way forward names; `ReconcileOrigin` goes on with the other branches and never returns an error for it.
The pair's code branch move goes ahead, since it changes no marker and may bring the warp a commit that tracks the new anchor.
`BranchState.stops` counts `BranchAnchorStopped`, so its entry reads stopped, and `stopsRemoval` and `stopsEntries`, the two CLI copies of that list, count it too.
The other switches over these states name it as well: `cleanupcli`'s and `adoptcli`'s `sharedStopText` render a prime-weft `anchor_stopped` as a shared stop with its way forward and no re-entry clause, since the `--set` is the step; `adoptcli`'s `stoppedBranches` lists an `anchor_stopped` branch, so an owned-elsewhere slug's owner write stops over it as over a diverged one; `adoptcli/doc.go`'s list of those states names it.
`battencli`'s `judgeOrigin` lets a cancel through an entry stopped only by `anchor_stopped` branches, while every other stopped branch still refuses: the `--set` the stop names refuses a worktree with an unfinished run and names `lyx batten cancel`, so the cancel is the exit that clears the run before the `--set`; the cancel's own records commit stays local until the `--set` integrates, as the stop's push does.
`stopWayForward` for it names the empty marker's file and way forward, or `lyx fabric anchor <slug> --set <Anchor>` run in the pair's code worktree; for the prime it names first ending every reed session of the hub, the orch's included, with `lyx reed down` in each, and waiting until no batten run of the hub is live, then the `--set`, then `lyx orch start` again.
`internal/fabricengine/doc.go`'s origin reconcile section lists the new state.

Tests:
`TestReconcileOrigin_Moves` gains a subtest: origin's pair weft and origin's prime weft each carry a new marker; a hub-scope moving reconcile reports both `anchor_stopped` with the way forward (the prime's naming `lyx reed down` and `lyx orch start`), leaves both wefts' `HEAD` and their `.lyx` unchanged, still fast-forwards the pair's code branch, and returns no error; a run-scoped reconcile of the pair reports the same; an empty own marker in a pair's weft is `anchor_stopped` naming `.lyx-anchor`.
`TestAdopt` gains rows on its fake report: a prime records branch `anchor_stopped` refuses with its way forward and no re-entry clause; an owned-elsewhere slug with an `anchor_stopped` weft stops before its owner write, naming the branch and its state.
`TestCancelRun` gains rows on its fake report: an entry stopped only by an `anchor_stopped` branch is cancelled; one stopped by an `anchor_stopped` and a `diverged` branch still refuses.

**ImpactSummary:** one new branch state threaded through the mover, its way forward and every CLI switch over the states (`fabriccli`, `hibernatecli`, `cleanupcli`, `adoptcli`, `battencli`'s cancel); no existing state's handling changes.
