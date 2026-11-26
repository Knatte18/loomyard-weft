---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: "refuseNothingLeft now counts a pending record whose Rerun is this cancel's command (TeardownPending in cancelcmd.go), so the cancel reaches removePairAndBranch with only the origin records branch left; TestCancelIntegration_FinishesAnUnfinishedRemoval drives that state and asserts the branch is deleted."
  - id: F2
    status: resolved
    answer: "The doc.go paragraph now names only fabricengine behaviour (the caller's rerun in job and record, the `lym fabric remove` default, SettleGonePairTeardown), and each statement matches the code."
  - id: F3
    status: resolved
    answer: "F1's fix makes the cancel finish in the no-branch-on-origin state (pending record counted, ErrPairNotFound settled), and the comment now states that condition; TestRemove_PendingRecordNamesTheRerunItsCallerGave pins the unconditional wording."
  - id: F4
    status: resolved
    answer: "Settled by the added entry \"A removal record names the cancel only when a cancel wrote it\"."
  - id: F5
    status: resolved
    answer: "removePairAndBranch calls SettleGonePairTeardown on ErrPairNotFound; it pushes owed archive tags under the teardown lock and drops the record. The second cancel in TestCancelIntegration_FinishesAnUnfinishedRemoval and the second settle in TestRemove_PendingRecordNamesTheRerunItsCallerGave drive the gone-everywhere state and assert PendingTeardowns lists none."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
    answer: "Subtest RemoveWhoseWaitRunsOutNamesTheGivenRerun asserts the ErrPairPushRunning refusal names the given rerun and not `lym fabric remove`."
  - id: F8
    status: resolved
    answer: "The closure now calls removeThrough, and TestRemoveThrough_HandsTheRequestToTheOriginRemoval fails if Request.Rerun is dropped."
  - id: F9
    status: resolved
---

No new BLOCKING finding on the changed hunks; every prior finding is resolved.

## Focus directives

- **rerunWayForward / failedRemoteSteps against refuseNothingLeft / removePairAndBranch.**
  Records branch only on origin: TeardownPending is true, so the cancel runs `Remove(..., remote=true, rerun)`, which finds the origin branch through refuseWhenNothingRemains and archives and deletes it.
  In production the handed-off child then clears the record; in a test binary the in-process hold never clears it, which is why the integration test's record survives the first re-run.
  Nothing left anywhere: Remove refuses with ErrPairNotFound before any lock, the cancel calls SettleGonePairTeardown, which takes the lock, rechecks origin, pushes owed tags from both stores and clears the record.
  A failed-step record keeps `Rerun` (pendingTeardownOf copies it), so TeardownPending matches it too.
  An end-only cancel with a pending record no longer refuses up front, ends nothing and returns the same "run again once origin answers" outcome as before.
  Verdict: the named command finishes the teardown in every listed state.
- **recordsArchiveWayForward with a rerun.** With origin holding no branch and the cancel's records and claim cleared, the cancel still proceeds on the pending record and the settle pushes the WeftTip tag if the hand push was skipped.
  The way forward and its comment are true.
- **removePair ErrPairNotFound and clearPendingTeardown.** The record is retired by the settle, and both tests named above assert `PendingTeardowns` lists none afterwards.
- **doc.go paragraph.** Documents only fabricengine; every statement holds.
- **PATTERN-pair-remote guard text.** The count now matches `removeArgCount = 5`.
  The line still omits that the guard skips `_test.go` files and the two owning packages, an omission the base text already had (see below).
- **Rerun reaching its destinations.** Both new tests fail if the rerun is dropped at Remove's lock refusal or in the pairteardown forwarding.
- **summary.md.** The What-changes bullets match the diff for fabric, pairteardown and battencli, and the blast radius names all three.

## Observations below the incremental threshold

This round admits only BLOCKING new findings, so these are recorded for the parent, not raised:

- **Merge danger door.** `SettleGonePairTeardown` adds a path that pushes archive tags to origin, and the rubric holds a `two-way` call against a diff that pushes outward.
  The tags are additive and idempotent, so a revert leaves only extra archive tags; I judge this MEDIUM at most.
- **Settle scope.** `SettleGonePairTeardown` drops any pending record of a gone pair whatever its `Rerun`.
  A cancel with `--remove` can therefore also settle a record written by `lym fabric remove` or dobby, which the added entry places out of scope.
  It is harmless (owed tags are pushed first); matching `record.Rerun == rerun` would keep it to the cancel's own records.
- **Integration test doc.** The doc of `TestCancelIntegration_FinishesAnUnfinishedRemoval` says the record "still stands" after the first re-run.
  That holds only because a test binary runs the remote steps in process; in production the child clears it.
- **Pattern guard text.** `pattern/PATTERN-pair-remote.md` says the guard flags a five-argument `Remove` in any file importing fabricengine.
  It skips test files and the owning packages, and every test file now makes such calls.
- **Uncovered probe failure.** The cancel fake's `TeardownPending` never returns an error, so the "the unfinished teardowns" probe refusal has no test.
- **dobbyengine test.** The `scheduler.Stop()` added to `TestLandNotice_ReachesAnOrchStrandListedAfterTheLanding` is a gate fix outside the answer key, and summary.md does not mention it.
  `Stop` is idempotent, so the later cleanup is safe.
