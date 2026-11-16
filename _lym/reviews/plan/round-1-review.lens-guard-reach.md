```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row a card adds, I traced whether a real entry point can reach it.

**Per guard, reachable or not**
- **Card 2, `Act`:**
  - **Rules fed by `PushOwed`, `OriginUnreachable`, `CodeTipMissing` and the hold input:** not reachable in this part; every caller leaves these inputs at zero, as the Goal's Exports allows. Only `TestAct` reaches them. Deferred by the Goal, so no finding.
  - **`Aside.Cancel` and `Aside.Hold`:** not reachable. No card has `lym batten cancel` ask the act rule. Finding 6.
  - **Refusal reason for a kept `cancelled` run (`lym fabric remove`):** reachable through `lym loom resume` and `lym tender resume`. Not reachable through `lym batten run`, because `checkRunStatus` refuses first. Finding 7.
- **Card 3, `battenPreRun` act check:**
  - `wait` for `cancelling` and `hand_over`: reachable.
  - `wait` on the pause intent: reachable only once this machine's pause entry is gone, since `refuseCovered` runs first.
  - `landed`: shadowed by `checkRunStatus`'s done refusal, as the card itself says.
  - Fresh start: Derive matches no state, so it answers `act`, which is harmless.
- **Card 4, act check in `resumeFromRunState`:**
  - `wait` for `cancelling` and `cancelled`: reachable.
  - `hand_over`: not reachable. `resumeCmd`'s `refuseOwnedElsewhere` already refused it. Finding 2.
- **Card 5, `planPass` asking the act rule:**
  - `wait` for paused and `cancelling` (from the pushed intent): reachable.
  - `landed`: not reachable, because `drivePasses` drops done runs first. Harmless.
  - The "past its merge under a hold gets a pass" row: not reachable, since `planPass` never sets the hold input. Finding 5.
  - The cancel window before the intent is committed: left unguarded. Finding 3.
- **Card 6, the cancel's refusals:**
  - A live second cancel (lock held) and a dead writer (an intent with no finish): reachable.
  - `registeredBattenAlive` reading the intent: reachable.
  - Offline end-only: reachable.
  - `TestCancelIntegration_AnInterruptedCancelReadsCancellingOnAnotherClone`: no named way to stop the cancel at the point the test needs. Finding 8.
- **Card 7:**
  - The `FinishCancel` no-op on re-run, and an unpushed finish naming the re-run: reachable.
  - The synchronous push of the `--remove` record removal: pre-empted by card 9's commit. Finding 4.
- **Card 8, cleanup:**
  - Refusal while the pair stands: reachable, since that is the normal kept-cancel outcome.
  - Removal once the pair is gone: reachable through a crash fixture.
  - The new candidate also reaches `clearDoneRun` with no guard. Finding 1.
- **Card 9, deletion in `RemovePair`:**
  - Reachable for `cancel --remove` (an unfinished intent) and for `lym fabric remove` of a kept cancel. A Worktree-Teardown row with no intent correctly does nothing.
  - Skipped when the removal returns `ErrPairNotFound`; card 8 covers that leftover record.
- **Card 10, `lym tender pause` and `lym tender resume`:**
  - The owner refusal on pause: reachable when this machine's pair is owned elsewhere.
  - `ResumeAct` refusing `hand_over`: reachable. Its way forward cannot lift this machine's entry. Finding 9.
- **Card 11:**
  - A publish error failing the row: reachable.
  - The `resumeSession` fallback to a fresh session: has a reaching input but no named test. Finding 10.
- **Card 12, refusal of an empty `AttemptsDir` in `NewFinalize` and `NewParentAlign`:** a fail-loud wiring guard, so it is exempt.

**Findings**

1. **BLOCKING · consistency · card 8 (`internal/battencli#judgeStatus`)**
   Card 8 makes a finished `cancelled` record a candidate in `judgeStatus`. `clearDoneRun` calls `judgeStatus` at `internal/battencli/cleanup.go:222` to decide `done`. So `ClearDoneRun`, the dobby's done-run clear, would then take a cancelled record and discard its task branch on origin (`cleanup.go:237-241`). That breaks D3, which says the done-run clear never takes a cancelled run, and it breaks card 7's claim and its `TestClearDoneRun` row. Card 8 guards with `PairPresent` only in `cleanupSlug`; the same site is unguarded in `clearDoneRun`, which card 8 does not list.

2. **MEDIUM · design · card 4 (`internal/loomcli#loomCLI.resumeFromRunState`)**
   `resumeCmd` runs `refuseOwnedElsewhere` (`internal/loomcli/resume.go:276`, `ownercheck.go:103-126`) before it calls `resumeFromRunState`. That uses the same `runOwnerVerdict` the card feeds into `Inputs.OwnedElsewhere`. So the act rule's `hand_over` refusal, and the `StartOwnedElsewhereKind` envelope the card keeps for it, can never be reached in production. The `TestResumeVerb` row reaches it only by calling the method directly. The card also adds a second origin reconcile on every resume.

3. **MEDIUM · consistency · card 5 (`internal/dobbyengine#planPass`)**
   Card 5 drops the `run.CancelMarker` check because the verdict "covers" it. But the verdict reads only the batten record's intent.
   - Between card 5 and card 6, no intent exists, so the dobby sees no cancel at all.
   - After card 6, the window before the intent is committed is still left out. In that window only the local lock shows the cancel, and card 6 folds the lock into `RunView.CancelMarker`, which `planPass` no longer reads.

   The Shared Decision says this machine's driving verbs see the cancel in that window, but `planPass` is no longer guarded there. The spawned pass would still refuse inside `lym batten run`, and the dobby would count that as a failing pass. `drivePasses` still reads `run.CancelMarker` (`internal/dobbyengine/passes.go:203`).

4. **MEDIUM · consistency · cards 7 and 9 (`internal/battencli#clearRunPhase`, `internal/pairteardown#Teardown.RemovePair`)**
   `cancel --remove` calls `deps.RemovePair` = `teardown.RemovePair` (`internal/battencli/cancelcmd.go:286`, `cancel.go:717`). Card 9 makes that step delete and commit the record and spawn a detached push. Card 7's following `RemoveRecords` then finds nothing tracked and commits nothing (`cleanup.go:486-495`), so its "pushed at once through the same publish" path has no commit to push. The removal reaches origin only through card 9's detached `SpawnStatusPush`. `TestCancelIntegration_RemoveTearsDownPairAndBranch`'s assertion "record gone from origin when cancel returns" therefore depends on a detached process.

5. **LOW · design · card 5 (`internal/dobbyengine#TestPlanPass`)**
   The row "a run past its merge under a hold gets a pass" cannot be reached through `planPass`. The card leaves the hold input at zero, and `run.Held` counts as a pause, so a run past its merge under an adopt hold gets `wait`. The row only pins a test-only input combination.

6. **LOW · scope · card 2 (`plan:internal/runfsm#Aside`)**
   `Aside.Cancel` and `Aside.Hold` are named "for `lym batten cancel`", but no card has the cancel ask the act rule (cards 6 and 7 never call `runfsm.Act`). Those aside branches are reachable only from `TestAct`.

7. **LOW · consistency · card 1 with card 2's `Verdict.Refusal`**
   `lym batten run` of a kept cancelled record goes through `checkRunStatus` (`internal/battencli/arm.go:530-540`) before the act check. That check refuses `cancelled` as an unrecognized state, or as done if card 1 gives it the `StateDone` branch (`arm.go:531` is one of card 1's grep hits). So the act rule's "`lym fabric remove &lt;slug&gt;` for a kept cancelled run" way forward is never reached from batten run. Instead the operator gets either a refusal with no way forward or `doneSlugRefusal`'s done-run wording.

8. **LOW · design · card 6 (`plan:internal/battencli#TestCancelIntegration_AnInterruptedCancelReadsCancellingOnAnotherClone`)**
   The tier1 test "stops a cancel after its intent is published", but `cancelRun` has no seam a real-hub test can use to stop between the publish and the end. The card names no input that brings the cancel to that state.

9. **MEDIUM · design · card 10 (`internal/tenderengine#Resume`, `ResumeAct`)**
   `PauseOwnedElsewhere` writes `OwnerCaller` entries on exactly the runs another machine owns (`internal/tendercli/deps.go:233-250`). After card 10, `lym tender resume &lt;slug&gt;` of such a slug answers `hand_over` and refuses. Its way forward is "the same verb on the owner machine, or `lym adopt` here". The first lifts nothing on this machine, because the entry is machine-local. The guarded state occurs routinely, and the named way forward does not clear it.

10. **MEDIUM · design · card 11 (`internal/landingshed#PRReview.resumeSession`)**
    The card prescribes a new branch: start the PR-Review session fresh when the recorded session's transcript is absent here. Its reaching input is a record written on another machine, but no test reaching that branch is named. The rubric requires one.

Tool calls: none; the inherited context held everything this lens needed.
