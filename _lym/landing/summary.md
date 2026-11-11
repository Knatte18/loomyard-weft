# Dobby frees the slot of a run whose entry ended and tears down its pair

A run that landed could still hold a `max_runs` slot until someone ran `lym cleanup --apply` by hand.
`reed-slack-relay` landed and its board entry went done, but its batten run stopped as `failed` at Run-Shed, before Worktree-Teardown.
Dobby counted that failed run under `max_runs` for six days, so a released task waited behind it.
Nothing removed its pair either.

Dobby now gives no slot to a run whose board entry is done or gone, whatever batten's state says.
When batten leaves such a run short of its own teardown, its dobby tears the pair down itself through the hub's origin reconcile, as `lym cleanup --apply` does.

## Changes

- `internal/dobbyengine`
  - `RunView` gains `EntryEnded`, set when the run's board entry is done or gone from the board, and `PairPresent`.
    `slotsTaken` leaves out `EntryEnded` runs.
    The start step and `BuildView` both count through it, so the view's `max_runs` reasons change with it.
  - New `ended.go`: a run is stranded when its entry ended, its pair is on disk and dobby gives it no pass.
    That means batten's state is neither running, nor paused with its work on its parent, nor done.
    A run with a pass in flight, a held driver lock, a cancel marker, a covering pause or an owned-elsewhere mark is not stranded.
    After the passes, each tick hands every stranded run whose wait ran out to one call of the new `Deps.TearDownEnded` seam.
    The call returns a channel, and a later tick reads the result.
    Only one teardown is in flight at a time.
  - The log gets `teardown <slug>: pair removed`, or `teardown <slug> failing: <reason>` once per failure episode.
    A failed run waits one probe interval, then twice its last wait after each further failure, up to an hour.
    The waits are kept in memory only.
  - A teardown in flight keeps the loop at the probe cadence and holds the binary check, so a re-exec cannot cut it short.
- `internal/battencli`: `DobbyRuns` sets `EntryEnded` from the local board, through the new `entryEnded`.
  It sets `PairPresent` from whether the task worktree's path exists.
- `internal/beamcli`: `hubDobbyDeps` wires `TearDownEnded` for every dobby to `tearDownEnded`.
  That runs `fabricengine.ReconcileOrigin` over the hub scope from the prime, with `pairteardown`'s hub seams, in a goroutine.
  `endedTeardownResults` reads the report: a run counts as removed only when its ended entry says `Removed`.
  A busy pair, a failed teardown, an unreachable origin, a reconcile error or an entry origin does not show ended each give their reason.
- The `dobbyengine` and `beamcli` package docs describe the slot rule, the ended teardown and its wiring.

## Design decisions

- The teardown goes through the hub-scope origin reconcile rather than a direct `pairteardown` call.
  The reconcile removes a pair only when origin's board shows the entry ended, the same check `lym cleanup --apply` relies on.
  So dobby never removes a pair on a local board read alone.
  The run scope cannot be used, because it never tears down its own run's pair.
- The reconcile is hub-wide: besides the stranded run, it tears down every other pair origin has ended,
  and it moves the hub's branches behind origin, as `lym beam start` and a hibernate wake already do.
  It runs only when a stranded run exists, and then at most once per wait.
- The slot rule reads the local board, but a teardown does not rely on that read alone.
  A run whose entry reads done locally frees its slot at once,
  while its pair goes only once origin agrees.
- The failed batten run record stays after the teardown.
  It holds no slot and is no longer stranded once its pair is gone.
  Clearing it is still `lym batten cancel <slug> --remove`, as the removal's follow-up already says.
- No notice goes to the orch when a teardown keeps failing; the dobby log carries it.

## To check by hand

- No real dobby ran this teardown: the dobby tests drive it through a fake seam, and `endedTeardownResults` is tested on built reports.
  On a hub with a landed run left `failed` before Worktree-Teardown, check that `lym beam status` no longer counts it under `max_runs`.
  Then check that the dobby log shows `teardown <slug>: pair removed` within a probe interval or two, and that the pair is gone.
- `finalizeEntered` in `internal/fabricengine/replay.go` drops a run this machine's own landing reached from the ended set.
  If a stranded run is dropped that way, the reconcile leaves its pair.
  The dobby then logs `origin's board does not show <slug> ended` and retries up to hourly.
  The `reed-slack-relay` refusal shows its reconcile did judge that run ended, but confirm on a live case.
- The hub reconcile also tears down a beam's own pair when origin ended that beam.
  When a beam dobby triggers it, that teardown ends the dobby itself partway through.
  The next attempt or `lym cleanup --apply` finishes it; this is a rare case and is only documented, not handled.
