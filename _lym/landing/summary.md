# Dobby frees the slot and pair of a run whose board entry ended

A run whose board entry is done or gone could be left stopped by batten before its Worktree-Teardown row.
Dobby counted that halted run under `max_runs`, so it held a slot until someone ran `lym cleanup --apply` by hand.
`reed-slack-relay` held a slot for six days this way and kept a released task waiting.

Dobby now gives such a run no slot and tears down its pair itself.

## What changes

- `battencli.DobbyRuns` sets two new `dobbyengine.RunView` fields.
  `EntryEnded` is true when the run's board entry is done or no longer on the local board.
  `PairPresent` is true when the run's task worktree is still on disk.
- `slotsTaken` in `dobbyengine/starts.go` no longer counts a run whose `EntryEnded` is set, whatever batten's state says.
  `max_runs` therefore frees as soon as the board entry ends.
- The new `dobbyengine/ended.go` picks the stranded runs.
  A stranded run has an ended entry and a pair on disk, and dobby gives it no pass.
  Runs with a pass in flight, a held driver lock, a cancel marker, a covering pause or an owned-elsewhere mark are left alone.
- Each tick hands the stranded runs whose wait ran out to the new `Deps.TearDownEnded` seam, after the passes.
  Only one teardown is in flight at a time.
  A removed pair logs `teardown <slug>: pair removed`.
  A failed teardown logs `teardown <slug> failing: <reason>` once per episode.
- `beamcli.tearDownEnded` fills the seam.
  It runs `fabricengine.ReconcileOrigin` over the hub scope from the prime, with the seams `pairteardown` builds, as `lym cleanup --apply` does.
  It reads each requested run as removed only when the report's ended entry for it says so.
  An unreachable origin, a busy pair and a failed teardown each come back as a reason.

## Design decisions

- The teardown reuses the origin reconcile instead of a narrower per-run removal.
  The reconcile also moves the hub's branches behind origin, as `lym cleanup --apply` does, so one tick can do more than the stranded pair.
  A run origin does not show as ended is not torn down, and its reason names `lym batten status <slug>`.
- After a failure a run waits one probe interval, then twice its last wait, up to an hour.
  A removed run waits one probe interval too, because the listing of the tick that read the result still shows its pair.
  The waits are in memory only, and a run that is no longer stranded forgets its wait.
- While a teardown is in flight, the loop skips its idle backoff and the binary check.
  A re-exec would cut the teardown short, half done.
  A nil `TearDownEnded` tears nothing down.

## To check by hand

Nothing here was run against a real hub.
- With a run whose entry is done and which batten left `failed` before Worktree-Teardown, start a dobby and check that its slot frees at once and its pair goes within a probe interval.
- Check that a failed teardown logs once and backs off, for example with origin unreachable.
- Check that the hub's branches the reconcile moves behind origin are the ones `lym cleanup --apply` moves.
