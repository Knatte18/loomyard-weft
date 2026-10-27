# Fixer report — plan, round 2

## Changes

- **F1** (MEDIUM, design), fixed as proposed, with one difference in where `stopAt` is stated: card 8's row "a worktree removed after tick 1 has ended" now asserts `len(h.ticks) == 1`, that the worktree path does not exist after `Run` returns, and the one `StopStrand`.
  `CheckBinary` is kept only as the check that nothing re-execs.
  The closing clause now says a second tick, or a recreated `.lym/beam` stub, fails the row.
  `stopAt` 3 is stated once, in the harness sentence that opens `TestRun_WorktreeGoneExits`, for every beam row, not in this row alone; see the self-check.
- **F2** (MEDIUM, design), fixed differently in one part: card 8's kept-tidy paragraph now reads the record first and then tests `worktreeGone` once.
  When gone, the read and its error are discarded and the loop exits with the earlier `keptTidy`; the paragraph says the rows that remove inside a tick reach this branch with a failed read, since the `Tick` seam is `daemon.tick`'s last step.
  When not gone, a non-decode read error is logged through `d.failures` and the earlier `keptTidy` is kept; "ends the loop as a tick error does" is gone.
  The new row deviates from the proposal: it does not do the later removal in a later tick's hook.
  Once `state.json` is a directory, tick 2's `UpdateState` (`internal/dobbyengine/loop.go`, `daemon.tick`) fails before the `Tick` seam runs, so no later hook call could remove the worktree.
  The row instead removes it through the same `Deps.Sleep` wrapper as F1's row, and asserts that `Run` did not exit after tick 1 (`CheckBinary` called once) and that the start record's `Tidy` is the one stopped.

F3 is decision-class and left to the parent.

## Self-check

- fix-affected-set, MEDIUM: `stopAt` was named only in the two `Deps.Sleep` rows, and other beam rows could be masked by the seam's cancel. Fixed as proposed: `stopAt` 3 is in the harness sentence for every beam row and has been removed from the two rows.
- fix-consistency, LOW (`stopAt` on the other rows): duplicate of the fix-affected-set finding, fixed with it.
- fix-consistency, LOW (decode failure and absent record branch unstated): fixed as proposed. The not-gone branch now says that a record that is absent or fails to decode sets `keptTidy` to nil.
- fix-proposal, LOW F2-b (decode-failure branch): duplicate of the one above, fixed with it.
- fix-consistency, LOW (new row does not override the default watch event): fixed as proposed. The row's tick-1 hook now sends no watch event.
- fix-proposal, LOW F2-a (new row's watch event can hang the `Deps.Sleep` send): duplicate of the one above, fixed with it.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
