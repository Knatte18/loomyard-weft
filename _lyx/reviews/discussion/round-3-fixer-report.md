# Fixer report — discussion round 3

All changes are in `_lyx/discussion/decision-record.md`.

## Changes

- F1 (BLOCKING), fixed as proposed, using the review's primary option.
  D7 now says batten adds no restart: it sends its existing driver-dead notice, and the operator restarts the chain with `lyx loom start` or `lyx loom resume`.
  The batten acceptance criterion now pins the notice instead of a revival.
  The self-check added that the notice waits until the owner pause is lifted (`internal/battenshed/doc.go:28`).
- F2 (MEDIUM), fixed differently.
  The proposed sentence was "exits only for a hub entry".
  It would still defer a Finalize's Done push under a hub entry, such as a pending hibernate, and the lift spawn excludes a pair owned elsewhere, so nothing would ever respawn that push.
  D6 now says a child whose slug holds a parent lock record never exits for coverage: it keeps the gate's waiting behaviour, passing a slug entry and waiting out a hub entry.
  The acceptance clause pins both cases: the push is not deferred, and under a hub entry it waits at the gate until the lift.
- F3 (MEDIUM), fixed as proposed, with a correction.
  D4 states the order for a pair whose task worktree is gone.
  The quiet wait, both run-lock waits and the loop end are skipped as they are today.
  The driver-lock re-check is skipped because it is new in this record, not "as today", and none of these steps can read a gone worktree (`internal/pairteardown/teardown.go` `EndSession`, under `if !gone`).
  The refusal probe still runs, and `End` comes before the session end by exact name.
  An acceptance criterion was added for the gone-worktree case.
  `PATTERN-pair-teardown`'s change line now limits "never its run-lock wait" to a pair whose task worktree is present.
- F4 (LOW), fixed as proposed: D4's Bound replaced.
- F5 (LOW), fixed differently.
  D3's `End` is now bounded after SIGKILL and returns an error naming each survivor's pid and role, beside the processes it ended.
  A zombie counts as gone.
  D3's engine-constant line and its rationale now say that `End` returns without an error only once the pushes are gone.
  D8: `lyx tender reap` reports survivors and exits non-zero.
  The proposal was that an `End` error refuses every removal.
  That would contradict D4's rule that batten's Worktree-Teardown and the reconcile's forced teardown always go on, and it states no outcome for `lyx reed down` or the D7 sites.
  D4 now says:
  - `RefuseWhenBusy` callers and `lyx reed down` refuse, with nothing removed, the session still live, each survivor's pid named and the way forward of retrying once it is gone.
  - Batten's Worktree-Teardown and the forced teardown go on and report the survivors in `SessionResult`.
  - The D7 sites log the survivors at Warn and leave the pause standing.

  The new `End` acceptance criterion pins each of these outcomes.

## Self-check

- fix-consistency, MEDIUM, gone-pair order leaves the run-lock waits open against D5 and `PATTERN-pair-teardown`: fixed as proposed (merged with fix-claims MEDIUM).
- fix-consistency, LOW, `End` error has no outcome for `reed down`, the D7 sites, or `ended_processes`: fixed as proposed (merged with fix-way-forward LOW).
- fix-consistency, LOW, driver-dead notice is withheld while the owner pause covers the slug: fixed as proposed (merged with fix-claims LOW).
- fix-claims, MEDIUM, gone-pair sentence omits the run-lock waits and calls the driver-lock re-check "as today": fixed as proposed.
- fix-claims, LOW, notice comes only once the owner pause is lifted: fixed as proposed.
- fix-way-forward, LOW, the `End` error refusal names no way forward: fixed as proposed (retry once the pid is gone).
- fix-way-forward, LOW, acceptance row pins no way forward: fixed as proposed.
- fix-proposal, MEDIUM, a zombie push of a spared spawner reads alive and would refuse batten's teardown: fixed as proposed (a zombie counts as gone; non-`RefuseWhenBusy` teardowns go on).
- fix-proposal, MEDIUM, F2's sentence still defers a Finalize push under a hub entry: fixed with own change (the parent-lock child keeps the gate's waiting behaviour), so F2 is marked fixed differently above.

## Disputed

None.

## Deferred

None.
