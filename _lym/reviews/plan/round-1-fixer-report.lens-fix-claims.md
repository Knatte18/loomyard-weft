```yaml
lens: fix-claims
outcome: reported
```

Checked the claims the fix added or rewrote against the code at HEAD.

**1. MEDIUM — card 10 contradicts its own loomcli row**
- Changed text: card 10 Intent, "Every other reader of the kind, and those two for another party's unsettled merge, names only `lym fabric merge --abort` … and `loomCLI.refuseOverUnfinishedMerge`, which refuses naming the abort." The fix also added the `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` row "the row's own unsettled merge-in at Finalize proceeds".
- Problem: "those two" covers only `ownMergeInLeftover` and `preMerge.clear`. So the Intent has `refuseOverUnfinishedMerge` refuse every unsettled merge, including the row's own, which the new row says proceeds.
- Evidence: `internal/loomcli/start.go:236-246` calls `ownMergeInLeftover` only inside `case fabricengine.MidMergeParked`. Widening `ownMergeInLeftover` alone does not reach the new kind; `refuseOverUnfinishedMerge` needs the same own-leftover case for `MidMergeRecordsUnsettled`, and the Intent says the opposite.

**2. MEDIUM — the takeover's merge route can deadlock on the locks**
- Changed text: card 10, "`takeoverBranch.commitEntry` takes the same route in that state, under the write lock, which it takes for the route".
- Problem: `commitEntry` runs inside `takeoverBranch.land`, which already holds the push lock (`internal/fabricengine/takeoverwrite.go:177`). Taking the write lock there with a blocking acquire reverses the order the code uses elsewhere.
- Evidence: `lockWeftForReplay` takes the write lock and then the push lock, both blocking (`internal/fabricengine/pull.go:504` then `:509`). Code that already holds the push lock only try-acquires the write lock (`internal/fabricengine/pushclassified.go:715`, `TryAcquireWriteLock`). As written, a takeover and a replay running together can deadlock. The card names no acquire order and no try-acquire.

**3. MEDIUM — card 7's "only off that branch" contradicts card 7's own key text**
- Changed text: card 7, "A one-repository prime on its recorded branch already holds the marker card 5 committed, so the marker commit reaches a prime only off that branch, where it now refuses".
- Problem: the fix also added that `lym.primeBranch` "lets records commits land on whichever branch it names … an operator who sets it to another branch moves the prime's records there". Such a branch need not carry `.lym-anchor`. `migrateWeftMarker` and `recordAdoptedWeftAnchor` commit whenever HEAD lacks a committed marker (`internal/fabricengine/reconcile.go`, the `migrateWeftMarker` body; `anchormarker.go`, the `recordAdoptedWeftAnchor` body). So the marker commit can reach a prime that is on its recorded branch, unrefused and possibly mid-merge. The stated bound does not hold.

**4. LOW — the reason given for the retry is wrong**
- Changed text: card 7, "wrap the `commitAnchorMarker` call in `retryHeldIndexLock`, whose re-run commits nothing more since the marker's bytes are then committed".
- Problem: `retryHeldIndexLock` re-runs only after the commit failed on a held `index.lock`. At that point nothing was committed, so the re-run does commit. The sentence's reasoning is false. The conclusion that no commit is duplicated holds, but only because the failed attempt committed nothing.

**5. LOW — battencli's doc does not use the phrase card 11 names**
- Changed text: card 11, "The docs of `loomcli`'s and `battencli`'s `newCommitStatusSeam` name the wait where they name skip-while-mid-merge".
- Problem: only loomcli's doc says "skip-while-mid-merge" (`internal/loomcli/wiring.go:339`). battencli's says "It skips while a merge is in progress" (`internal/battencli/commitstatus.go:204`), so the instruction does not locate battencli's sentence.

Tool calls:
1. Grepped `refuseOverUnfinishedMerge`'s use of `ownMergeInLeftover` and every write-lock and push-lock acquisition in fabricengine. My context did not have that function's switch or the repository's lock order.
2. Read `internal/loomcli/start.go:222-262`, `internal/fabricengine/pull.go:498-512` and `internal/fabricengine/pushclassified.go:705-720`, to confirm the case structure and the order the two locks are taken in. My context had only summaries of these.
