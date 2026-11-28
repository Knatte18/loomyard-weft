```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the plan in `_lym/plan`: for each decision, criterion and branch, which card carries it and which planned row reaches it.

1. **BLOCKING — D2/D4, card 7 (`joinQueue`): the entry's `TrueMerge` is read from the wrong operand.**
   - Card 7 sets `TrueMerge` only when `PullRequest.Mode` reads `merge-in`.
   - Today's landing shape is `Squash: (fz.deps.Config.Squash || landing.atOnce) && !landing.trueMerge` (`internal/landingshed/finalize.go:564`). So on a hub with `landing.squash: false`, every run true-merges.
   - The entry should record the shape that run's own parent-side merge would take. As planned, a batch on a non-squash hub writes single-parent squash commits.
   - No row covers a non-squash hub's batch. Card 3's true-merge row drives `BuildLandingChain` directly, so it does not reach this.

2. **MEDIUM — D5, owner checks resolve the task branch ref: no card carries this.**
   - D5 says "`ParentHoldsTaskWork` and the owner checks resolve the task branch ref".
   - Card 4 changes only `parentHoldsTaskWorkAt`. Card 8 changes only the loomcli `TaskHead` seam.
   - The owner checks run through `Deps.Reconcile` (`runReconciler`, `internal/loomcli#landingDeps`), `ParentOwner` and `OwnedElsewhere`, and no card names those for a detached worktree. No row runs the owner checks with the leader detached.

3. **MEDIUM — D5, criterion "a run resumed at any row … is put back on its task branch first", card 10: no planned test reaches the production wiring.**
   - The `Shed.Restore` closure lives in `internal/loomcli#loomCLI.newShed`.
   - `TestFinalizeQueue_ResumedDetached` is in `landingshed_test`. It cannot call that unexported closure, so its "after the shed restore the loomcli closure runs" row cannot run as written.
   - `TestShed_ResumeRestoreRunsBeforeAnyRow` drives only a fake hook.
   - No row resumes at a row other than Finalize, and no loomcli test pins that `newShed` sets `Restore`.

4. **MEDIUM — D5, the pusher holds off the code side while the marker stands, card 4: the `pushClassified` hold-off has no row.**
   - Card 4 makes `internal/fabricengine#pushClassified` push nothing on the code side of a marked worktree.
   - Only `TestRunCommitsAhead` and `TestRefusedPushStands` gain rows. Neither drives a push, so a broken hold-off passes.

5. **MEDIUM — D5, task-head reads on the batch path resolve the branch, card 8: the `TaskHead` change has no row.**
   - Card 8 changes loomcli's `TaskHead` to read `refs/heads/<taskBranch>`.
   - No row reads `TaskHead` with the worktree detached, as `takeBatch`'s moved-member check and `judgeMember`'s re-read do. A HEAD-reading `TaskHead` would pass every planned row.

6. **MEDIUM — D6, the rebuild cap, card 8: no row reaches it.**
   - D6: "One lock hold makes at most `landing.batch_max` such rebuilds … past that the leader lands alone".
   - No row drives repeated moved heads or left-out members past the cap.

7. **MEDIUM — D6, the records-commit fallback to the entry's records head, card 8: no row reaches it.**
   - The fallback to `RecordsHead` when `RecordsToHold` fails has no row.

8. **MEDIUM — D7, runner crash or memory-pressure stop on the batch gate, card 9: no row reaches it.**
   - Card 9 says such a stop halts the leader and sets no outcome. No `TestFinalizeQueue_BisectsAFailedBatch` row drives an unjudged batch gate result.

9. **MEDIUM — D7/D4, the batch gate's flaky callback, cards 8 and 9: no row.**
   - The rule is: log at Warn with the chain commit, and add nothing to `fz.flaky`.
   - No row makes the batch gate's first run fail and its rerun pass. Nothing asserts the leader's landing message carries no flaky section.

10. **MEDIUM — D8, a run with outcome `failed` or `conflict` leads no batch, cards 7 and 8: no row.**
    - Card 7's rows run before card 8 adds batches.
    - Card 8's skip row checks only that a leader skips another's entry that has an outcome. No row has a `failed` or `conflict` run take the lock while other live entries wait, and assert that it takes no member.

11. **MEDIUM — D9, criterion "a hub default layer setting `landing.finalize_lost_races` loads with the unknown-key warning", card 6: the warning is not asserted.**
    - `TestLoadConfig_TemplateDefaults` asserts only that the file loads with the template's values.
    - The criterion names the warning. Nothing asserts the local and machine layers refuse the key either.

12. **LOW — D5, a darn leader's batch gate covers every member's trees, card 8: the row is in a different package than the criterion names.**
    - The row is `internal/loomcli#TestDarnTreeScope_DiffsTheDetachedChainCommit`; the criterion names `internal/landingshed`.
    - It pins the scope function, not a leader's batch gate running those targets.

13. **LOW — D8/D6, "a run whose `landed` commit the parent's head no longer holds … leads a batch", card 7: the row asserts only that the run lands.**
    - Card 7's row asserts "merges, gates and lands". After card 8, no row asserts that such a run leads, taking other members.

14. **LOW — `ownLandingCommit`'s last fallback, card 7: no row.**
    - With neither the entry's commit nor `NewestLanding`, the parent's head is recorded with no landed size and the landing read as not empty. No row reaches this.

15. **LOW — branches no planned row reaches:**
    - **Card 1:** `Backstop` writes a ledger landing for a mark member with no entry.
    - **Card 3:**
      - a non-SHA `base` or task head errors naming the slug;
      - `MoveCodeTo` refuses a detached head;
      - `NewestLandingOf` stops at its 500-commit cap.
    - **Card 4:** `CheckoutBranch` refuses a dirty checkout, and a marker that names no existing branch is an error.
    - **Card 7:**
      - `joinQueue` writes no entry when `needsGitHubOnDisk` fails to read;
      - a failed `Queue.Join` returns an error;
      - a failed `Queue.Remove` is logged at Warn and keeps Done.
    - **Card 8:**
      - a failed `MemberDeps` is skipped with a Warn;
      - right before the move, the liveness re-read leaves out a member whose owner died;
      - `judgeMember` leaves out a member on its `LandBlockers` or `ParentOwner` read.
    - **Card 9:**
      - the marker stands across bisection steps;
      - a culprit leader runs no second gate on the prefix-1 tree.
    - **Card 10:** `detachedStop` restores a clean marked worktree that reaches it with no shed restore before it.
    - **Card 11:** a backstop error releases the lock and is returned.

16. **LOW — card 9, `runFixer`'s new entry point: the Intent is unclear and no row reaches the claim.**
    - The Intent says the fixer "skips no check of its own but spends the attempt from the recorded log".
    - That does not say whether the culprit's fixer reruns the gate before it starts, which D7 forbids ("no gate runs twice on one tree").

No further tool calls were made.
