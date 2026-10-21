# Card 20 — cleanup-reconcile

**Create:**
- `plan:internal/cleanupcli#cleanupSeamOptions` -> `func cleanupSeamOptions(prime *lyxcwd.Location) pairteardown.SeamOptions`

**Edit:**
- `internal/cleanupcli#Command`
- `internal/cleanupcli/doc.go`
- `internal/cleanupcli#TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover`

**Uses:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/pairteardown#HubSeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams`
- `internal/hibernateengine#PauseCaller`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D8's `lyx cleanup`: it runs the hub reconcile before its segments.
After the prime check, `Command`'s run calls `fabricengine.ReconcileOrigin` over the hub scope, `JudgeOnly` unless `--apply`, with the seams of `pairteardown.New(prime).OriginSeams(cleanupSeamOptions(prime))`, and puts the report on the envelope as `reconcile`, beside the top-level `mutations` (the report's `Mutations.Entries()`, always an array, empty in the dry run) and `partial` (true exactly when the envelope is an error and the record is non-empty, as `fabriccli`'s envelope derives it) pair PATTERN-mutation-record requires of a mutating verb, since `--apply`'s reconcile removes pairs, writes snapshot tags and moves branches.
`mutations` carries the reconcile's record only; the segments keep reporting their own removals under their existing keys, unchanged by this card.
Without `--apply` the reconcile judges only: both ls-remote calls, a fetch of the moved heads' objects into no ref, no move, no replay and no teardown; a diverged Bolt or prime records branch is reported as needing a replay and nothing is refused, and the report names each move and teardown that accounts for `--apply`'s segments listing fewer.
With `--apply` the moving reconcile runs first and the segments compute their candidates from the reconciled state; a D3 stop (on Bolt or the prime's records branch) refuses the verb with the report and runs no segment, naming the stop's way forward, the stopped branch's `BranchOutcome.WayForward`, followed for a dirty, gone or busy write-lock stop by the re-entry `then re-run lyx cleanup --apply from the prime` (card 15); a pair stop is only reported; an `Unreachable` report is a warning and the segments run; a reconcile error refuses `--apply` naming it and runs no segment, and in the dry run is printed with the segments run.
Under an operator's live hub pause the reconcile waits at the network gate until the lift.
`cleanupSeamOptions` extends `HubSeamOptions` with D6's wake hold: `WakeHeld(slug)` reads the pause record for a live entry of the slug whose caller is `hibernateengine.PauseCaller`, and `LiftWakeHold(slug)` lifts that entry by identity through tender's `Lift`, so `--apply`'s teardown of an ended entry the wake held ends its driver, snapshots its uncommitted paths under a local `set-aside/` tag and lifts the hold (card 18), and a re-run after a parked push released the teardown lock removes it.
The long help and `doc.go` describe the reconcile step, the dry run's judge-only bound (the dry run moves no ref, tracking refs included) and the refusal.

`TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover`, the verb's existing scenario test over a `hubforge` hub, calls `boardkit.InstallReplay` and gains steps, with a second clone of the records bare as the other machine: the dry run over a diverged Bolt prints the report with Bolt needing a replay, exits ok with `mutations` an empty array and `partial` false, and moves, replays and tears down nothing (every ref unchanged); `--apply` with a tracked change on a path the Bolt replay touches refuses with the report and runs no segment, its refusal text containing Bolt's `WayForward`, the dirty line's Bolt form exactly, followed by `then re-run lyx cleanup --apply from the prime`, and so does `--apply` over a both-sides change of the prime weft's records, its refusal text containing `recordsConflictWayForward`'s exact line with `lyx fabric push` in the prime's worktree; with the records repo's origin URL set to `http://127.0.0.1:1/x` (refused, a transport failure) both modes warn `unreachable` and run the segments; with the records repo's origin URL set to a path that holds no repository, which `ReconcileOrigin` returns as an error since it is not a transport failure, `--apply` refuses naming the error and runs no segment, and the dry run prints the error and still runs the segments; a pair stopped as diverged is reported and the segments run; `--apply` over a run whose records origin deleted removes it so it is not listed as paused afterwards; `failed_pushes` (card 10) is on both envelopes.
A further step runs D6's force case end to end over an ended entry whose pair holds a wake-written pair pause entry (caller `hibernateengine.PauseCaller`) and a dirty task worktree: with the pair's teardown lock held by the test, standing in for a parked push, the first `--apply` snapshots the uncommitted paths under a local `set-aside/` tag, lifts the pause entry and reports the pair busy, not removed; after the test releases the lock, a second `--apply` finds the snapshot tag (`EarlierSnapshot`), passes force over the still-dirty worktree and removes the pair; the first `--apply`'s envelope `mutations` holds the snapshot tag and the second's holds the removed pair; a segment failing after a reconcile that moved a branch sets `partial` true.

**ImpactSummary:** The cleanup verb gains a reconcile step, one refusal and the `mutations` and `partial` keys; its segments and their output keys are unchanged.

**Commit:** 20: cleanup runs the hub's origin reconcile, judge-only without --apply
