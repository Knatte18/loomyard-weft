```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked each planned test row against the tests, fakes and helpers it runs on.

**Findings**

1. **BLOCKING** — Card 4, card 5 (`ProtectedMarkerPushLeavesNothing`) and card 14 (`ProtectedRefusalRecordsProtected`): the planned pre-receive hook cannot produce the `protected` class.
   - gitrepo wraps a rejection as `ErrPushRejected` plus git's rejection line only (`internal/gitrepo/push.go:194`, `:265`, `lastLineHolding` at `:271`). Any line the hook prints itself is dropped.
   - A plain pre-receive hook's rejection line ends in `(pre-receive hook declined)`. That holds neither marker (`protected branch`, `repository rule violations`), so `ProtectedBranchRefusal(err)` reads false.
   - The record then lands as `not_fast_forward` or `other`. For the existing hook rows, `pushclasses_integration_test.go:335` asserts exactly `(pre-receive hook declined)`.
   - Each of the three rows fails as written. Either the gitrepo push error must also carry the `remote:` message lines (a gitrepo edit no card lists), or the rows need another way to produce the refusal.

2. **MEDIUM** — Card 4: a protected refusal is retried through the whole rejection backoff before it is classified.
   - `ErrPushRejected` goes to `settleRejection`/`repushOverOriginContained`, and `rejectionRetryKind` (pushclassified.go:198-216) stops on none of the protected markers.
   - The card puts the check only in `classifyFailure`, which runs after the retries. It lists no edit to `rejectionRetryKind`, and no row asserts the retry count for this class.

3. **BLOCKING** — Card 1, row `SuffixedCheckoutBesideIt`: the expected anchor `.` is wrong.
   - The linked worktree is created with `worktree add -b` from a branch where `.lym-anchor` (`backend`) is committed, so the suffixed checkout carries the root marker itself.
   - `OneRepository` reads true there, the anchor reads `backend`, and a cwd outside `backend` is refused. The row fails.
   - It needs a checkout whose branch lacks the marker, such as an orphan branch, to test the "resolves as itself" path.

4. **BLOCKING** — Card 10, `MergeStateOperation`/`MergeStateActive`: an existing row of `TestMergeStateActive` breaks.
   - The listed sources are `MERGE_HEAD`, `CHERRY_PICK_HEAD`, `REVERT_HEAD` and the rebase directories, and `MergeStateActive` becomes `operation != ""`.
   - The existing row "weft conflicted squash without MERGE_HEAD reports true" (`mergestateactive_integration_test.go`, the `driveConflictedSquashMerge` shape) then reads false.
   - The card's "as today" claim fails unless a conflicted index (`ConflictedFiles`) is named as a source.

5. **BLOCKING** — Card 10, row `CrashBeforeStepThreeCommitsAbortsAndPushes`: the hook cannot stop the merge.
   - The row needs "the card 9 hook returning a crash", but card 9 declares the hook `func()` (`SetAfterMergeStartForTest(hook func())`), which returns nothing.
   - Nothing lets the hook end `Merge` between steps 2 and 3 with the record left unsettled. No card lists a hook that returns an error, or another way to stop there.

6. **MEDIUM** — Card 7, row `RecordsAndCodeCommitsRunOneAfterTheOther`: the row cannot fail when the code is wrong.
   - It asserts that a `Fabric.Commit` "does not land until the hold is released". Not having landed yet holds at any moment even with no lock taken, so the assertion races and cannot fail deterministically. PATTERN-test-speed's "Time is injectable" section requires an outcome signal instead.
   - `commitBothSides` already serialises code and records under one lock today (`commit.go:207-220`). The behavior that changes is the lock's location from card 3, which `TestRecordsGeometry_OneRepository` already asserts. The row repeats existing behavior.

7. **MEDIUM** — Card 11: no row reaches the real wiring of `MergeOperation` or `Protected`.
   - Card 11 drops the two real-merge rows of `TestCommitStatusSeam_Real` because the seam would sleep through `time.Sleep`. Both shed seams hard-wire `Sleep: time.Sleep` (`newCommitStatusSeam`).
   - The ordinary-path rows only see `""`, so a miswired `MergeOperation`, or a missing `Protected` field in `commitStatusDepsGuardedBy`/`battenCommitStatusDeps`, still passes.
   - Fix: add a `Sleep` field to `commitStatusDeps` that a row can drive (for example, abort the merge on the first sleep), and keep a real waits-then-commits row.

8. **MEDIUM** — Card 12, row `NoCodeTipIsOwed`: it passes without the change.
   - `ReturnToPreviousTransition` already returns false when origin's records branch holds fewer than two transitions of the slug (codetipsettle.go:64-66).
   - Unless the row pushes two transitions first, it cannot fail. The card states no transition count.

9. **MEDIUM** — Card 12, row `TagsPushedBeforeTheBranchDeleteAndTheViewRefGoes`: it can pass vacuously.
   - No code in this part publishes a `lym-view/` ref (the next part does), and the row does not say it plants `refs/heads/lym-view/<slug>` on origin first.
   - Without that, "no view ref" holds even if `deleteViewRefOnOrigin` never runs.
   - Its order claim (tag before branch delete) also has no stated observation point.

10. **MEDIUM** — Card 13: the new `FastForwardPair` refusal has no row.
    - `FastForwardPair` gains "refuses with `FastForwardNotFastForward` only when a local non-records commit is unpushed", and no row of `TestReplay_OneRepository` reaches it.
    - The new owed stop in `retryWeftPush` (deferred with `Commits`) and the diverged report in `reconcileRecordsBranch` are also unreached. Only the pull stop has a row (`LocalCodeCommitRewritesNothingAndStopsNamingIt`).

11. **MEDIUM** — Card 7, `refuseOffPrimeBranch`: its gate does not stop two-repository primes.
    - It applies when "its gitdir equals its common dir". In a two-repository hub the prime's weft sibling is also a main checkout (a clone), so the gate holds there too.
    - Since `lym.primeBranch` is absent there, every two-repository prime records commit would refuse.
    - The Intent must state the one-repository condition (`OneRepository`). No row asserts that a two-repository prime still commits; existing tests would catch it at the gate rather than on the card.

12. **LOW** — Test economy: three new rows repeat or rest on existing table rows.
    - Card 4's "`FailedPushesOwedBy` leaves out a `protected` record" and card 14's "lists an `operator_unpushed` record" each repeat what `TestFailedPushRecord/OwedByListsTheRunsOwedClassesNewestFirst` (failedpush_test.go:116) already reaches. They belong as records added to that row, not new rows.
    - Card 4's protected row pins behavior the card says it leaves unchanged.
    - Card 4's `TestIsPushOwed` row `ProtectedJoinedWithTransport` asserts not-owed, which the existing `OwedJoinedWithRejection` row (pushowed_test.go) already reaches for a non-owed class. Only the `wantProtected` half is new.

13. **LOW** — Card 12 and the fixture: `AddPairWith` (hubforge/hub.go:424) pushes through `CoalescePushBothAt(res.Path, RecordsWorktreePath(...))`.
    - With card 3 both paths are the same worktree, so the one-repository rows push the one branch twice as "records" and "code".
    - Card 12 edits `coalescePushSides` but never says that `CoalescePushBothAt` with equal paths pushes once. The `PushesTheBranchOnce` assertion ("records one branch push") depends on it, so the card should state it.

**Tool calls (3)**
- Read `pushowed_test.go`, the head of `mergestateactive_integration_test.go` and the `MidMergeKind` consts, for the rows cards 4, 10 and 14 extend; my context had only their names.
- Grepped `TestCommitStatusSeam_Real` steps, the `MergeActive` users in loomcli/battencli and the `pushclasses_integration_test.go` hook setup, for card 11's dropped rows and the hook mechanics cards 4, 5 and 14 rely on.
- Grepped `internal/gitrepo/push.go` rejection wrapping, the `TestFailedPushRecord` subtests and the backoff helpers, to check whether a hook's message reaches `ProtectedBranchRefusal`.
