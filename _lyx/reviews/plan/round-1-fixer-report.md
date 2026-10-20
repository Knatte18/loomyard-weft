# Fixer report — plan, round 1

Every finding's premise was checked against the code and holds; all 19 are fixed.

## Changes

- **F1** (fixed as proposed): card 7 adds `internal/shedrecipe#newTestEnv` and `internal/testkit/envkit#FullEnv` to **Edit:**, and its test paragraph names the four `LandDeps` test literals; the overview's verify line gains `./internal/shedrecipe ./internal/testkit/envkit`.
- **F2** (fixed differently): card 4's `finalizeEntered` no longer counts `current_producer` `Finalize` alone.
  It counts a `Finalize` history row, a `Finalize` current producer whose local parent branch already holds the task's work by `Fabric.HoldsWork` (Finalize's parent-side merge makes this true before `markTaskDone`, `internal/landingshed/finalize.go:226-229`), or the prime batten status at `Land` or later.
  The proposed "evidence Finalize writes once it has pushed its done mark" was replaced by the merge evidence, which already exists.
  The proposed card 22 row "over a real status file" was not made: card 22's tests drive a fake `Reconcile`, so such a row would never reach `finalizeEntered`. The end-to-end rows went to card 17's `TestReconcileOrigin_Ended`, which runs the real reconcile over a `hubforge` hub.
  Card 4's `TestRecordsReplay` row and Uses (`Fabric.HoldsWork`, `ReadOriginFor`) and card 22's prose were updated to match.
- **F3** (fixed as proposed): cards 21 and 22 name `lyx cleanup --apply` from the prime, whose hub reconcile runs D6's teardown, and their ended test rows assert it.
- **F4** (fixed as proposed): card 15 creates `gitexec#RunWithin` and `gitexec#runCoreWithin`, edits `gitexec#runCore` to delegate with `remoteDeadline`, and states that `lsRemotePair` calls `RunWithin`, with the gate hold before the deadline; `./internal/gitexec` added to verify.
- **F5** (fixed as proposed): card 9 drops `goneBackoffSleep` and `SetGoneBackoffSleepForTest`, edits `SyncOptions` with a `BackoffSleep` field, adds `opts SyncOptions` to `pushClassified`'s signature, and rewords the test paragraph to set `opts.BackoffSleep`.
- **F6** (fixed as proposed): card 13 creates `commitRecordsPathsHeld`, edits `CommitRecordsPaths` to lock and delegate, and states that `commitPendingRecords` with `try` true try-acquires the lock itself.
- **F7** (fixed differently on the test): card 13's local-mode list skips `respawnPairPush`.
  The proposed "through an existing hook" was not usable: `removeAfterTeardownHook` runs after the worktrees are gone and returns nothing (`internal/fabricengine/remove.go:286`, `teardownlock.go:30`). Card 13 instead creates `removeAfterLockHook` with `SetRemoveAfterLockHookForTest`, and its `TestRemoveLocal` row fails a removal through that hook while both worktrees survive.
- **F8** (fixed as proposed): card 9 edits `PushWarpRebaseFreeAt`, routes it through `pushClassified` on the code side, and adds a row for a rewound task branch pushed through `Fabric.PushBranch`.
- **F9** (fixed as proposed): card 18 makes `SeamOptions.HubPaused` a `func(slug string) (bool, error)` that is read at each `LivePair` call, and both tests gain a lifted-after-build row.
- **F10** (fixed as proposed): card 5 edits `cmd/lyx#TestStencilSeeding_HubScenario`.
  The existing step now replays.
  A new conflicting step keeps the skip and asserts the Warn names `notes/card.md` and `lyx board sync`.
- **F11** (fixed as proposed): card 1's `FailedPushWayForward` now depends on side and class.
  It adds a `bolt` side (`lyx board sync`) and three helpers (`recordsConflictWayForward`, `codeDivergedWayForward`, `rewoundWayForward`).
  `FailedPush` gains the paths, commits and origin head that the lines name.
  `gone` names `lyx fabric remove` and, once the entry has ended, `lyx cleanup --apply`.
  The test asserts one row per side and class pair a push writes.
- **F12** (fixed differently): the proposal's "keeping this machine's by settling the named paths then `lyx fabric commit`/`lyx board sync`" was not used, because that push replays again and meets the same conflict (the review's own premise, `internal/boardengine/sync.go:53`).
  The records-conflict line instead names `git merge origin/<branch>` in the worktree, settling and committing, then the side's push verb, whose push is then a fast-forward.
  The other choice is `git reset --keep origin/<branch>`.
  Rewound names `git reset --keep <origin head>` or `git merge --strategy ours`, each then `lyx fabric push`.
  A diverged code branch names merging the remote task branch.
  Cards 5, 6, 9, 15 and 16 use the helpers, and their existing test rows assert the exact lines.
- **F13** (fixed as proposed): card 3's `Apply` loads the board config with `LoadConfig(configengine.Dirs{Board: ...}, "board")` (without the skip env), sets the store vocabulary and the `RenderToDisk` outputs from it, and adds a row for a label known only through `board.yaml`.
- **F14** (fixed as proposed): card 9's transport bullet records an `IsTransportFailure` error at once and `ErrPushRetriesExhausted` after the backoff, and its row drops "after the backoff".
  Card 8's twin sentence was reworded too (see the self-check).
- **F15** (fixed differently after the self-check): card 17's ended-busy outcome and card 15's `Summary` now name the way forward.
  For a busy driver, the removal waits for the driver to end, which card 22's Halt at Merge-In or Finalize makes certain, and then `lyx cleanup --apply`.
  For a held lock, `lyx cleanup --apply` once the lock is released.
  The proposed `lyx tender pause <slug>` was not used: it interrupts turns but leaves the driver strand alive (`internal/tendercli/cli.go:82-95`), so `probeQuiet` would still read busy.
  The test row asserts the line.
- **F16** (fixed as proposed): card 21's reconcile-error way forward names fixing the cause and then `lyx batten run <slug>`.
  Card 22's stopped Stuck appends `lyx loom resume` in the task worktree.
  The rows assert both.
- **F17** (fixed as proposed, second option): card 9 states that `Fabric.PushWeft` and `PushWarpAt` stay on `PushCoalesced` because only tests call them.
- **F18** (fixed as proposed, first option): card 9 creates `PushFailedError` with `Error` and `Unwrap`, returned by `pushClassified` for a recorded failure.
  `writeHubRules` reads the class with `errors.As`, and `TestWriteHubRules`' fake returns one.
- **F19** (fixed as proposed): card 8 says the error wraps `ErrPushAuthRefused` with git's last error line, matched with `errors.Is`.

## Self-check

- fix-affected-set: no findings.
- fix-proposal, MEDIUM: the `BackoffSleep` func field makes `SyncOptions` non-comparable, which breaks `internal/landingshed/mergein_test.go:261`. Fixed differently: the field stays, and card 9 adds `internal/landingshed#TestMergeIn_SyncParent` to **Edit:**, rewriting the comparison field-wise.
- fix-claims, MEDIUM: card 8's open-risk sentence said a reworded 403 would be recorded as `transport`, but `IsTransportFailure` falls back to not transient (`internal/gitexec/transport.go:34`). Fixed as proposed: it now reaches card 9 unclassified.
- fix-claims, MEDIUM: card 17's `lyx batten pause <slug>` way forward is not shown to quiet the task's driver. Fixed differently: the way forward waits for the driver to end, which card 22's Halt makes certain; cards 15 and 17 updated.
- fix-consistency, LOW: card 17 and card 22 still used the loose "Finalize entered". Fixed as proposed: both now read Finalize past its parent-side merge, or a `Finalize` history row.
- fix-consistency, LOW: card 17's busy way forward disagreed with card 23's wake notice. Fixed as proposed: card 17 states that under a wake-written hold the force case ends the driver, so the bare re-run applies.
- fix-consistency, LOW: card 13's ImpactSummary was stale. Fixed as proposed.
- fix-consistency, LOW: card 4's Uses lacked `ReadOriginFor`. Fixed as proposed.
- fix-consistency, LOW: the overview's index line for card 1 said "per class". Fixed as proposed.

## Disputed

None.

## Deferred

None.
