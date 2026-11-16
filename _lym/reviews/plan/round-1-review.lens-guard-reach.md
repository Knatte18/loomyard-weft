```yaml
lens: guard-reach
outcome: reported
```

guard-reach: per guard, reachable or not

**Card 2/6, `ActRefusal` reasons**
- `owned_elsewhere`, `pause`, `paused` (intent), `paused` (shed), `cancelling` (live), `cancelling` (none), `cancelled`, `landed`: reachable through `AskAct` from cards 12, 13 and 14.
- `push_owed`, `code_tip`, `hold`: not reachable. In this part `AskAct` leaves `PushOwedOffline`, `CodeTipMissing` and `HoldCovers` false (card 6 Intent; overview Shared Decisions). Only `TestActRefusal` reaches these wordings, and it calls only the constructor.

**Card 3, shedengine refusals of a cancelled status**
- `Goto`, `Relaunch`: reachable through `lym batten goto`. `startGuard.refuse` reads only an unfinished intent as present, so a finished cancel gets through to them.
- `Shed.stepLocked`: not reachable from batten's entry points once cards 10 and 12 land. `battenPreStepUnmarked` (card 10) and `refuseByAct` in `battenPreRun` (card 12) refuse a cancelled record first. Before card 10 nothing writes `cancelled` at all. It is a defensive engine guard.

**Card 4, `UpdateRunRecord` absent-record branch**: reachable (a pause or cancel of a pair that has no batten record).

**Card 9**
- `takeCancel` refusing a held lock: reachable.
- `readCancel` torn-record branch: reachable only through a planted file.
- Push-did-not-land refusal: reachable when the push is rejected or the lock is busy.
- `refuseNothingLeft` counting a present intent: not reachable as a deciding term. The intent lives inside the run's records directory, so a present intent implies `RecordsPresent` is already true.
- `PublishIntent` on a slug with no batten record: reachable, and the card does not guard it.

**Card 10**
- `judgeStatus` cancelled + pair stands → skip naming `lym fabric remove`: reachable.
- `judgeStatus` cancelled + pair gone → candidate: reachable after a crash or a failed drop.
- `clearDoneRun` leaving a cancelled run: reachable.
- `battenPreStepUnmarked` refusing a cancelled run: reachable through `lym batten step`.
- `dropCancelledRecord`: reachable through `lym fabric remove` (fabriccli → `pairteardown.New` → `Run` → `RemovePair`). Also reached inside `cancel --remove` itself.
- `BattenFollowUp` cancelled wording: reachable only when the drop failed.

**Card 11**
- `planPass` with a non-act verdict → no pass: reachable.
- Hand-over → `ownedElsewhere`: reachable.
- `slotsTaken` with a cancelled run: reachable, since a kept cancel's record and seed stay listed.
- Cancelling `Phase` in `strandedEnded`, `readWaits`, `runNotices`: reachable.
- `planPass`'s `if covered {…}` block: not reachable once card 11 lands. An act verdict implies no pause covers the run, and `Covered` and `PauseCovers` read the same record.

**Card 12, `refuseByAct`**
- Hand-over with a status found: reachable.
- Pause intent: reachable.
- Cancelled: reachable.
- Cancelling: not reachable. The upstream start-guard cancel check refuses first.
- Landed passing on to "the start guard": not reachable as described.
- A pause reaching the record after the covered check: reachable only by a race, and no test row covers it.
- Failed ask: reachable, with no test row (likely exempt as a raw I/O error).

**Card 13, `refuseByAct`**
- Cancelling, pause intent, act, failed ask: reachable.
- Hand-over: reachable only when `refuseOwnedElsewhere`'s judgment from origin disagrees with the stale local Bolt, which is exactly when the refusal is wrong.

**Card 14**
- `RunOwner` refusal: reachable.
- `RunAct` refusal: reachable, including for a landed or cancelled run.
- A slug-scope resume under a hub-wide pause: the existing refusal is reached only after `SetRunPause(false)` has already written and pushed.

**Guarded sites that recur unguarded**: `lym batten step`, `lym batten goto`, `lym loom start` and the generic `lym shed step` act on a run without asking the act rule.

## Findings

1. **MEDIUM · design · card 12 `battenCLI.refuseByAct`, cancelling branch.** This branch is unreachable from `lym batten run`.
   - `armAt` runs `refuseAtStart` (arm.go:347), which runs `startGuard.refuse`. That check refuses run, step and goto while a cancel is present (startguard.go:281-286; card 9 renames `Marker` to `Cancel`). This happens before the `PreRun` hook `battenPreRun` (arm.go:417, drive.go:84).
   - `armSeed` (arm.go:246) and `preDriveCheck` (arm.go:765) refuse it too.
   - `TestRefuseByAct`'s cancelling row reaches it only through a fake `askAct`. The upstream check is `startGuard.refuse`.

2. **MEDIUM · consistency · card 12 Intent, "A landed verdict passes: a done record goes on to the start guard".** The start guard has already run by then.
   - `refuseAtStart` and `claimStart`, the done-motif clear, both run in `armAt` (arm.go:347-354), before `battenPreRun`.
   - By `PreRun` a done record has already been cleared or refused, so the landed pass-through has no route to the start guard. The card's justification for setting a landed verdict aside is wrong.

3. **MEDIUM · design · card 13 `loomCLI.refuseByAct`, hand-over branch.** The upstream `refuseOwnedElsewhere` (ownercheck.go:103-126) runs first.
   - It judges the owner through the origin reconcile (ownercheck.go:60-72) and falls back to the local Bolt offline (ownercheck.go:64-65), the same source `AskAct` reads (card 6).
   - So the hand-over branch is reached only when the local Bolt is stale against origin, which is when the hand-over is wrong.
   - The card's test reaches the branch only with a nil `ownerOf` and a fake `askAct`.

4. **MEDIUM · consistency · card 9 `endRunPhase` with card 6 `PublishBattenRecord`.** The intent is published even when nothing was recorded.
   - `recordCancelIntent` reports false for a slug with no batten record. The card still has `endRunPhase` "record the intent and publish it". `PublishBattenRecord` then commits the run directory's path "when the record is gone".
   - For a slug whose run directory git never tracked, staging an unknown pathspec fails. loomcli/wiring.go:96-107 records this: "pathspec did not match any file(s) known to git".
   - That turns the cancel of a leftover pair with no records, which `refuseNothingLeft` lets through on pair parts (cancel.go:675-677), into a push failure that ends nothing.
   - The card states no skip of `PublishIntent` when `RecordIntent` reports false.

5. **MEDIUM · design · card 6 `pairteardown.ActRefusal` (owed-push, code-tip, hold wordings).** No input reaches them in this part.
   - `AskAct` keeps the three inputs false (card 6 Intent; 00-overview.md Shared Decisions "stay false in this part").
   - `TestActRefusal` calls only the constructor, which the rubric says does not reach a refusal.

6. **MEDIUM · design · card 12/13 scope versus D1 "Every verb or loop that acts on a run asks the act rule first".** The guarded site recurs unguarded.
   - `lym batten step` (`battenPreStepUnmarked`, arm.go:883), `lym batten goto`, `lym loom start` and `lym shed step` drive or move a run without asking.
   - So a paused-by-intent or owned-elsewhere run is refused by `lym batten run` and `lym loom resume` but still stepped by these verbs.

7. **LOW · consistency · card 14 `tenderengine.Resume` ordering.**
   - Resume calls `SetRunPause(slug, false)`, which writes and pushes, before the existing lift.
   - The lift can still refuse "has no pause of its own, and a hub-wide pause covers it" (resume.go:121-124). So the refusal is reached after the intent was cleared on origin, and "nothing is written" does not hold for that refusal.

8. **LOW · design · card 9 `refuseNothingLeft` "counts a present intent as left".**
   - The intent is a field of the status file inside the run directory, so `recordsHere` (cancel.go:658, 675) is already true whenever the intent is present. The added term never decides.

9. **LOW · design · card 3 `Shed.stepLocked` cancelled refusal.**
   - From batten's verbs, card 10's `battenPreStepUnmarked` and card 12's `refuseByAct` reject a cancelled record first, so the state does not reach `stepLocked` (arm.go:883-908).
   - `TestStep_CancelledRunIsNeverStepped` reaches it only by calling the engine directly.

10. **LOW · design · card 11 `planPass` `if covered` block (passes.go:144-152).**
    - With "no pass unless the verdict is act" and the act rule waiting on `PauseCovers`, `covered` is false whenever the block is reached. Its continue and lift branches become dead.
    - The card says only that "the continue pass under a covering pause goes", not that the block is removed.

## Tool calls
1. `grep` for `OriginRecordPathFor` and `OriginRecordPath` in fabricengine: I needed to know where the origin record lives to judge card 5's no-pair branch. It is in the weft worktree's durable directory, so it is readable from the records branch and no finding came of it.
2. `sed` of commitweftpaths.go lines 60-80: I needed the `CommitAnchoredPaths` contract for an unknown path to judge finding 4.
3. `sed` of startguard.go lines 300-330: I needed the rest of `startGuard.refuse` to see whether a found cancelled status is refused before `PreRun`. It is not, so the cancelled branch is reachable.
