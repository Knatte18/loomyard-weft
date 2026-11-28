```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch of the diff against concrete inputs.

**Function and branch list**

1. **`landUnderLock`, merge succeeds, head read succeeds** (`internal/landingshed/finalize.go:569-572`): run with head `L1`. `recordLanding` gets `L1` and `pushParent` gets `L1`. The base read the head twice under the same parent lock and got `L1` both times. Correct.
2. **`landUnderLock`, head read fails, `LandLocally` wired** (`finalize.go:570`, `finalize.go:1053-1054`): `recordLanding` logs and returns. `handedOverStop` runs first, as at base. Then `fail` is called with the same "read the head of parent branch" message the base produced. Correct.
3. **`landUnderLock`, head read fails, `LandLocally` nil or `PushSkipped`**: the error is ignored, `pushRecords` runs, and under `PushSkipped` `markTaskDone` runs. The outcome matches the base. One difference: the head is now read even when every seam is nil, where the base read it only when a seam needed it. That is one extra git read with no change in outcome. Correct.
4. **`recordLanding(commit, headErr)`, both seams nil**: returns early, as at base. Error not nil: logs the same message and records nothing. Correct.
5. **`pushParent`, ownership handed over**: `handedOverStop` returns before `headErr` is read, as at base. Correct.
6. **`landUnderLock`, pull request merged on GitHub first**: the parent-side merge then runs, and the head after it is the landing commit, as at base. Correct.
7. **`landUnderLock`, re-run after a partial landing (`held=true`)**: the merge commits nothing, and the head read after it is whatever the parent's head is now. The base did the same. But the comment the diff adds at `finalize.go:569` says this is "the run's own landing commit", which fails if another run landed on the parent between the two passes. That is finding 2.
8. **`writeLedgerLanding`, branch read succeeds**: writes `Landing{Run, Tip, RecordsCommit, ParentBranch, At}` through `runpusher.RecordLanding`, which replaces the run's standing landing (`internal/runpusher/parent.go:721-722`). Correct.
9. **`writeLedgerLanding`, branch read fails** (detached HEAD in the parent pair): returns a wrapped error and writes nothing. Correct inside the function itself, but see finding 1 for what its caller has already done by then.
10. **`writeLedgerLanding`, empty `records`**: stored as `""`, and `oldestHeldRecords` treats `""` as "no records commit" (`landingdeps.go:997`). Same as base. Correct.
11. **`recordHeldLanding`, success, resume (`committed=false`), and `oldestHeldRecords` keeping the earlier commit**: `records` flows unchanged into `writeLedgerLanding`. Same as base. Correct.
12. **`recordHeldLanding`, parent branch read fails**: this is a regression. The status commit is already made when the read fails. That is finding 1.

**Findings**

1. **LOW, design.** Location: `internal/loomcli/landingdeps.go:971` (`recordHeldLanding` → `writeLedgerLanding`).
   - Summary: the base read the parent pair's branch before it took the push lock and committed the run's status records, so a failed read left nothing behind. Now the read happens inside `writeLedgerLanding`, after `commitRunStatus`. A failure leaves a committed status checkpoint that no ledger landing holds back. Once the lock is released, a pair push can send it to origin before the landing is recorded, which the lock exists to prevent (`landingdeps.go:924-925`).
   - Recovery: a re-step recovers only if no pair push ran in between. That path finds the status already committed and takes the records head.
   - Context: the failures that were already possible after the commit (`oldestHeldRecords` and `RecordLanding`) leave the same state, so this widens an existing window rather than opening a new class of failure.
   - Fix: read the branch before the commit, as the base did, and pass it into the writer. This also matches the Exports line's "taking the parent branch".
   - Evidence: base `internal/loomcli/landingdeps.go` (`git show db892e0f9`), where `CurrentBranch` precedes `HoldPairPushLock` and `commitRunStatus`; the diff at `internal/loomcli/landingdeps.go:946-985`.
2. **LOW, design.** Location: `internal/landingshed/finalize.go:569`.
   - Summary: the added comment says the parent's head right after the merge is the run's own landing commit, and `recordLanding` and `pushParent` now pass it on under that name. Case 7 shows when that fails: a re-run with `held=true` whose merge commits nothing, after another run has landed since. Then the head is the other run's commit, and it gets recorded as this run's ledger tip and `RecordLanding` commit.
   - Context: the base behaved the same, so this is a false claim in the new comment rather than a new behaviour. Whether a parent can move between a run's merge and its Finalize re-run depends on when the parent lock is released, which I did not trace.
   - Evidence: `finalize.go:566-575` and `finalize.go:704-708` (`syncParent` skips when `held`).

**Tool calls**

- `git show db892e0f9` of `landingdeps.go` and of the `HeadSHA` calls in `finalize.go`: my context had only the diff, not the base order of the branch read against the lock and the commit, nor the base count of head reads.
