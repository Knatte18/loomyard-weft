# Fixer report — plan, round 1

Every finding's premise held against the code at HEAD.
`lym loom validate-plan` passes after the round, with only informational findings.

## Changes

- **F1** (fixed as proposed): card 5 adds `internal/fabricengine#pushClassified` under **Edit:**, and its Intent says `pushClassified` returns a `*TakeoverError` or an `ErrHandedOver` error before `classifyFailure` and `fail`, so no record or notice is written, and `pushSideDetached` returns it.
- **F2** (fixed as proposed): card 3 returns a records side that returned nil without landing as an error: a `deferred` `*PushFailedError` for a deferral, and an error naming the ended entry for an ended branch.
  The deferred row asserts that error.
  Card 8's `IsPushOwed` and card 7's `FailedPushesOwedBy` count class `deferred`, each with a new row.
  The overview's `owed-is-a-failed-push-record` decision, card 8's `Owed` bullet and card 1's `PushOwed` doc name the deferral too.
  Card 3's ImpactSummary names the new error.
- **F3** (fixed as proposed, then extended by the self-check): cards 6, 7 and 11 run the origin reset and the `git reset --keep` through executors in `internal/fabricengine/destroy.go` that check containment, ownership, dirtiness and force in that order, with ownership being the lease or the pushed archive tag.
  Card 6 also routes step 1's forward leased push through `updateRemoteBranch` and swaps `Repo.UpdateRemoteBranchLeased` in **Uses:** for `internal/fabricengine#updateRemoteBranch` (see Self-check).
- **F4** (fixed as proposed): card 8 adds the five tests under **Edit:**, and its Tests list says each replaces `Spawn` with `Push`, citing their lines.
- **F5** (fixed as proposed): card 3 returns `&PushFailedError{Record: <teardown_lock record>, Err: <wrapping ErrPairPushRunning>}`, and the held-past-wait row asserts both `errors.As` and `errors.Is`.
- **F7** (fixed as proposed, then corrected by the self-check): card 7 states that the reset drops every commit past the named tip, and states its three bounds.
  The first bound now holds only through the new `SyncOptions.AfterRecords` (see Self-check).
- **F8** (fixed as proposed): card 6 creates `OriginBattenStatus`, drops `OriginRunStatus` from **Uses:**, and the `droppedIntents` step reads through it.
- **F9** (fixed with my own change): card 9 drops `endRunPhase` from **Edit:**, replaces the end-only sentences with the proposed text, and drops the clause from the `TestCancelRun` row.
  The proposal deleted only from "`endRunPhase` now writes" onwards.
  I also removed the preceding clause "and today `endRunPhase` then stops with nothing written", which is the same false premise (`cancel.go:370-379`).
  I removed "lets an offline cancel write its intent" from the ImpactSummary for the same reason.
- **F10** (fixed as proposed): card 1's `code_tip` way forward names `lym fabric push` in the task worktree on `<CodeTipMachine>`, and drops that clause when the machine is empty, with rows in `TestAct`.
  Card 7's ImpactSummary says `lym batten cancel` waits on `code_tip` until the owing machine pushes or card 12 gives the tip up.
- **F11** (fixed as proposed, then refined by the self-check): card 12 pushes a missing tip that the local code branch holds through `PushRun` and gives nothing up.
  `codeTipDeps` gains `HoldsTip` and `PushTip`, and `TestSettleCodeTip` gains the row.
- **F12** (fixed as proposed): the overview's `out-of-part` decision names `Teardown.removeCancelledRunRecord`'s `SpawnStatusPush` as staying detached.
- **F13** (fixed as proposed): card 11's `markTaskDone` glyph carries `-> func (fz *Finalize) markTaskDone() error`.
- **F14** (fixed as proposed): card 3 moves `internal/fabriccli/doc.go` from **Uses:** to **Edit:**, and its Intent drops the doc's "push children it respawns" clause.
- **F15** (fixed as proposed, wording corrected by the self-check): card 8 says `ErrPushLockBusy` writes no record and its retry is owed only inside the seam.
- **F16** (fixed as proposed): card 6 adds `internal/fabriccli#addWeftVerbs` under **Edit:** and `internal/fabriccli/doc.go` under **Uses:**.
  Its Intent has `addWeftVerbs` build `MachineNames` through `pairteardown.ThisMachine`.

## Self-check

- fix-consistency, MEDIUM, card 8's "the holder lands the flagged sides on release" contradicts card 4's retired respawn: fixed with my own change.
  The proposal lens raised it at LOW and the claims lens at MEDIUM, so the three are merged.
  The sentence now says the seam's own retry lands its sides once the holder lets go of the lock.
- fix-consistency, LOW, card 1's `PushOwed` doc names two causes: fixed as proposed, by adding the deferred records side.
- fix-consistency, LOW, card 12's held-tip bullet does not say whether it runs inside the grace: fixed together with the proposal lens's F11 finding below.
- fix-claims, MEDIUM, card 7's first bound is false, because Finalize's parent push (`Fabric.PushBranch` → `PushWarpRebaseFreeAt`) reaches `repushOverOriginContained` with no records side first: fixed with my own change.
  `resetUnnamedTip` runs only for a code side sent with the new `SyncOptions.AfterRecords`, which `PushPairAnchored` and `coalescePushSides` set after the records side landed.
  Card 7 gains those three targets and a row in which a `PushWarpRebaseFreeAt` push keeps the rejection.
- fix-claims, MEDIUM, card 8's holder sentence: merged into the fix-consistency finding above.
- fix-proposal, MEDIUM, card 6 step 1's forward leased push stays a raw `UpdateRemoteBranchLeased` call outside `destroy.go`, which `cmd/lym/destructiveguard_test.go:80` bans: fixed as proposed.
  Both leased moves go through `updateRemoteBranch` (`destroy.go:1618`).
- fix-proposal, MEDIUM, F2's owed `deferred` class retries a dirty-replay deferral forever, with no way forward: fixed with my own change.
  I did not make the dirty-replay deferral not owed. Its record carries the same `deferred` class as a held-lock deferral, so `ReadRun` cannot tell the two apart without a new class, and `retryWeftPush`'s `(true, nil)` drops the cause.
  Instead, card 3 edits `pushJob.deferral`: over a worktree with uncommitted changes, the record's error names the worktree and says the changes must be committed or discarded.
  The `push_owed` refusal shows that error, and card 3 gains a row for it.
- fix-proposal, MEDIUM, F11 removes the grace bound for a held tip whose push keeps failing: fixed with my own change, using the lens's second option.
  Card 12 pushes a held tip inside the grace and past it, and never gives it up.
  Past the grace, a failing push logs a Warn naming its error and `lym fabric push` in this machine's task worktree.
  Card 1's `CodeTipGrace` doc says only a tip absent from the owner's repository is given up.
  The lens's first option, giving up a held tip past the grace, would undo F11's premise.
- fix-proposal, LOW, F15's sentence contradicts card 4: merged into the fix-consistency finding above.
- fix-affected-set, LOW, tender's hub-resume spawns (`tendercli.HubDeps`' `PushPair` and `PushPrimeRecords`, `internal/tenderengine/resume.go:247-254`) are neither moved nor named as staying detached: fixed as proposed.
  The overview's `out-of-part` decision names them as staying detached, since they push what was committed before the pause and write no transition of their own.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

- **F6** (decision class): the parent recorded the narrowing with `lym loom decision add` as the decision record's `Added after Discussion` entry "The owner's guard resets a code tip off the run's last transition's line" (creel #1494).
  The finding's change is to cite that entry in card 7's guard paragraph.
  Done: the paragraph now opens "The owner's guard (D2, as narrowed by the decision record's `Added after Discussion` entry …)".
  The paragraph already follows the narrowed rule, so the citation was the only change.
  I sent no flag to the parent.
