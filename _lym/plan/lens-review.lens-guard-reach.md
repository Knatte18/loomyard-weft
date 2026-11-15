```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens review of the plan's guards, refusals, branches and test rows.

1. **BLOCKING** (cards 5, 3, 14 and 10; decision later-part-inputs): `lym tender resume <slug>` can no longer lift an adopt pause, and every refusal over an adopt-paused run sends the operator to a wait that never ends.
   - Card 5 fills `Inputs.Held` from the pause record's adopt-pause coverage. `pauserecord.Record.CoverageOf` (internal/pauserecord/record.go:236-246) sets `Held` from a live entry of the slug's own scope that carries `AdoptCaller` or `HoldCaller`, and that same entry is what `lym tender resume <slug>` lifts.
   - Card 14's `Verdict` sets aside only `Pause` and `Cancel`. `Held` stays, so `Act` rule 5 answers wait with `ReasonHeld`, and `Resume` refuses with nothing lifted.
   - The same applies to card 10's `refuseByAct` for `lym batten run`. `Held` comes first in rule 5, so the refusal names "wait for the adopt that holds the hub to finish" instead of `lym tender resume <slug>`.
   - Today's way forward is that adopt pauses the run and `lym tender resume <slug>` lets it go on (internal/battencli/owner.go:106). PATTERN-dobby-runs says an adopt pause lifts only by hand. The new refusal therefore names a step that can never succeed from the state it leaves.
   - Fix: leave `Held` false in this part, since D7's Bolt holds fill it later. Or have `Aside.Pause` also clear a `Held` that came from the pause record.

2. **MEDIUM** (card 3 `Act` rule 4, `TestAct` row "act for a held landing past its merge"; card 5): the branch where a landing past its parent-side merge acts under a hold cannot occur in this part.
   - `Held` implies `PauseCovers`, because the adopt entry is a live entry of the slug's own scope and `Covering(slug)` is true (record.go:238-241).
   - Rule 4 requires `!PauseCovers`, so with card 5's inputs a held landing past its merge always waits.
   - D1's acceptance line "act for one landing past its parent-side merge under a hold" is reached only by the synthetic `TestAct` row.

3. **MEDIUM** (card 5, `PastParentMerge`): a run at batten's Land or Worktree-Teardown reads as before its merge.
   - Card 5 fills `PastParentMerge` only from `battenshed.FinalizePastMerge`. That returns false unless the child's `CurrentProducer` is Finalize and the state is running, or failed and transient (`battenshed/innerrun_owner.go:192`, from the earlier exploration).
   - Card 5 also reads `ChildLanding` true when batten is at `battenrecipe.NameLand` or Worktree-Teardown. Such a run therefore derives `Landing` with `PastParentMerge` false, although its parent-side merge has certainly happened.
   - Rule 4 is never reached for that stage, and so is the owner-finishes behaviour D1 and D4 describe.
   - `childReachedParent` (cancel.go:775-792) already counts these stages as reached. Card 5 should set `PastParentMerge` true in the same cases.

4. **MEDIUM** (card 10, D1 "every verb or loop that acts on a run asks the act rule first"): the rule is not asked on three other paths that act on a run.
   - `lym batten step` goes through `battenPreStepUnmarked` (arm.go:883-925) and `lym batten goto` through `battenPreGoto`. Neither gets `refuseByAct`, so step drives a paused or owned-elsewhere run with no act-rule check.
   - `lym loom start`'s resume of a parked driver is not touched by card 12, which edits only `resumeCmd`.
   - `startFirstPass` keeps `refuseCovered` alone (card 10).
   - No card adds the check at these sites or bounds their exclusion in an Intent.

5. **LOW** (card 10, `TestBattenPreRun_AsksTheActRule` "cancelling wait" row): the row cannot be reached through the real `lym batten run` entry.
   - `armAt` runs `refuseAtStart`, then `startGuard.refuse` and the marker reading, at persistent pre-run (arm.go:347). That happens before `battenPreRun`.
   - After card 8, "marker present" means a cancel intent with no finish. That is exactly runfsm's `Cancelling`, which the guard refuses first.
   - The act-rule refusal is reached only on a hand-built receiver.

6. **LOW** (card 12, `TestResumeVerb` rows "terminal landed wait" and the pause and hand-over waits): these branches are pre-empted by upstream checks or describe a state that cannot occur.
   - `resumeCmd` runs `refuseCovered` and `refuseOwnedElsewhere` before `runVerdict` (loomcli/resume.go:272-278). Pause-covers waits are always refused first by the pause record. A hand-over is reached only when the local Bolt disagrees with the judge-only origin reconcile.
   - `Landed` needs batten `done`, which comes only after Worktree-Teardown removed the pair, or board `done` with no batten record. In both cases no task worktree is left to run `lym loom resume` in.
   - While the batten record stands at Land, the run derives `Landing`.

7. **LOW** (card 7 `DobbyTasks` start refusal against card 3 `Derive` `Queued`): two readers give opposite answers for the same crash-leftover record.
   - Take a released task whose cancelled record outlived its pair, with the task branch gone. Card 3 derives `Queued` and `Act` answers act.
   - Card 7 makes `DobbyTasks` hold its start with `cancelledRunRefusal`, and `checkRunStatus` refuses it, until `lym batten cleanup`.
   - The FSM's `Queued` is therefore never acted on. The dobby-start decision reads the record's shed state instead of the FSM state, against D1's "every reader of a run's state takes it from internal/runfsm".

8. **LOW** (card 2 `WriteCancelIntent` "keeping a Finish already recorded only when intent's Remove matches it", card 1 `CancelFinish.Remove` true): this branch can never occur.
   - Under `--remove`, card 7 deletes the record and never calls `FinishCancel(true)`. A stored finish therefore always has `Remove` false.
   - A plain re-run over a kept finish is refused by card 7 step 2 before any intent write.
   - The "keep a matching finish" branch, and its `TestRunIntents` row if one is added, cannot occur.

9. **LOW** (card 2 `FinishCancel`, "with no Cancel recorded it records one from finish", and its `TestRunIntents` row): this branch cannot occur either.
   - Card 7 writes the intent before `EndForCancel`, and a slug with no record writes neither.
   - Every record that reaches `clearRunPhase`'s `FinishCancel(false)` therefore already holds the intent.

Tool calls beyond the inherited context:
- grep `Held` in internal/pauserecord: to confirm whether an adopt-held slug is also counted as pause-covered.
- grep `t.Run`/`RemovePair` in internal/pairteardown/originseams.go: to check whether the reconcile's pair removal bypasses card 9's `Run` hook. It does not, so no finding.
- sed of record.go:228-260 and grep of the `RemovePair` callers: the `CoverageOf` body and the removal call sites were not in context.
