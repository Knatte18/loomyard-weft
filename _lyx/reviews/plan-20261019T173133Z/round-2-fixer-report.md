# Round 2 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed in `_lyx/plan/09-creel-deliver-marked-probe.md`.
  The premise holds: `Store.do` calls the wake hook after it unlocks and before `post` returns (`internal/creelengine/pool.go`), and the worker registers `clock.After` only after a probe (`internal/creelengine/deliver.go`, `worker.run`).
  - `env.awaitPauseRead` is gone from `**Create:**` and its bullet is removed.
  - `awaitMarkedProbe` is now one loop under a real-time deadline and no longer waits on its own for the marked read.
  - `seen` is taken before the post and `mark` after it, with the reviewer's stated reason.
  - The Bound sentence now reads three passes, 24 s, for a lone worker.
  - The marker bullet says the post's wake runs before the post returns.
  - In the reshaping bullet, the caller never advances first.
- **F2** (LOW), fixed as proposed.
  The premise holds: `Scheduler.wake` starts a new worker with an empty wake channel (`deliver.go`).
  The reshaping bullet now says that helpers which post before `e.start` keep their `probeWaits(1)` wait and never call `awaitMarkedProbe`.
  The self-check below widened the wording beyond the two named helpers.

`lyx loom validate-plan` reports `ok: true` with only informational findings.

## Self-check

- fix-affected-set, MEDIUM: the post-before-start exemption named only two helpers, but many `TestScheduler_Reminders`/`TestScheduler_Injects` subtests post before `e.start` too.
  Fixed: the exemption is now stated by shape (a subtest or helper that posts before `e.start`), and the reshaping bullet's opening scope now reads "after a post made once `e.start` has run".
- fix-consistency, LOW: the two pre-start helpers stayed under `**Edit:**` although the card no longer changes them.
  Fixed: both are removed from `**Edit:**`.
  Neither changes: `orchStrandIsRemindedAndTypedInto`'s `tick` follows a pre-start post, so it races nothing.
- fix-consistency, LOW: "does not save that advance" lost its antecedent after the Bound rewrite.
  Fixed: it now reads "does not save the marked probe's pass".
- fix-claims, MEDIUM, merged with fix-proposal, MEDIUM: the 24 s bound conflicts with the kept per-check assertion against windows such as the 16 s stall freshness.
  Fixed with an own change: the assertion stays.
  A new sentence says a negative check whose shortest window is not longer than its bound cannot take the helper.
  The same sentence says no negative check this card reshapes runs under such a window, the shortest being `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry`'s 30-minute hold.
  I checked this against `deliver_test.go`.
  The post-after-start negative checks are `BusyReceiverIsRemindedAtItsNextIdleEdge` and `PauseTypesNothingUntilLifted`, which have no window, and the hold subtest.
  `TypedReminderClearsTheStall`'s second half is a positive journal check.
  The forks proposed dropping thresholds or reshaping checks differently; I made neither change, because no reshaped check meets the conflict.

## Disputed

None.

## Deferred

None.
