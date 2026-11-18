```yaml
lens: fix-affected-set
outcome: reported
```

Fix-check, affected-set lens: for each review finding whose premise is a set, I reran its search and checked that the fix covers every member.

Unchecked findings: F7, F8, F10, F11, F12, F13, F14 and F15. Their premises name a single site rather than a set, so no search was rerun. F1, F2 and F4 were checked against what the context already held: the only reader of the protected class is `IsPushProtected`, every `RecordsSettled` mention was renamed, and the context shows no mirror of `mergeState` besides `MergeStateForTest`.

**[MEDIUM] F3 — card 10: the own-leftover rule does not reach `loomCLI.refuseOverUnfinishedMerge`'s switch**
- **Changed text:** card 10, Intent: "Every other reader of the kind, and those two for another party's unsettled merge, names only `lym fabric merge --abort` for it: … and `loomCLI.refuseOverUnfinishedMerge`, which refuses naming the abort." The card also has the new row "the row's own unsettled merge-in at Finalize proceeds".
- **Problem:** `refuseOverUnfinishedMerge` calls `ownMergeInLeftover` only inside its `case fabricengine.MidMergeParked:` branch (`internal/loomcli/start.go:236-245`). Changing `ownMergeInLeftover` alone does not make an own `MidMergeRecordsUnsettled` merge-in proceed. The card text puts `refuseOverUnfinishedMerge` in the list of readers that always refuse, which contradicts the new proceed row.
- **Needed:** the Intent should say that `refuseOverUnfinishedMerge` handles `MidMergeRecordsUnsettled` the way it handles `MidMergeParked`, consulting `ownMergeInLeftover` before it refuses.

**[LOW] F9 — card 11: `failureDisposition`'s doc still describes the skip**
- **Changed text:** card 11 line 54, extended to cover the `newCommitStatusSeam` and `statusCommitPathspec` docs.
- **Problem:** `internal/statuscommit/statuscommit.go:147-155` (the `failureDisposition` doc) says "The mid-merge probe is not locked… That is the situation the mid-merge skip exists to absorb". Line 177 logs "skipping this transition". `failureDisposition` is on card 11's Edit list, but the card states doc changes only for the package comment and `New`'s doc, so this doc stays stale.

The other set members all check out:
- **F5:** the remaining direct `StageAndCommit` sites are the Bolt or the code side (`replay.go:913`, `bolt.go`, `shortnamecommit.go`, `stencilcommit.go`, `commit.go:225`).
- **F6:** the remaining `PushRebaseFree` sites are `retryWeftPush` (already guarded) and the Bolt push.
- **F3:** the `finalize.go:629` reader is `dirtyParentReason`, which is already covered.

Tool calls:
1. Grepped `StageAndCommit(` and `PushRebaseFree()` in fabricengine, to rerun the F5 and F6 searches, which my context did not hold.
2. Read `replay.go:895-920` and `pushclassified.go:765-862`, then grepped the `MidMergeParked` readers and the skip wording, to classify the F5 and F6 hits and rerun the F3 and F9 searches.
3. Read `battencli/commitstatus.go:125-150`, `loomcli/start.go:222-245`, `landingshed/finalize.go:610-635` and `battencli/wire.go:335-345`, and grepped for `mergeState` mirrors, to judge the F3 and F9 hits and rerun F4.
