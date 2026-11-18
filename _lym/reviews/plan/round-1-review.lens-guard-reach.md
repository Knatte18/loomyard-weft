```yaml
lens: guard-reach
outcome: reported
```

guard-reach: whether each guard, refusal and test row the cards add can be reached, and whether its guarded site recurs unguarded.

**Per guard, reachable or not**
- Card 1, `RootMarkerInvalid_IsErrorNamingFile` and `SuffixedCheckoutBesideIt`: reachable.
- Card 4, protected classification in `classifyFailure`: reachable. It is reached on every branch, not only the prime's records push (finding 1).
- Card 5, `--single-repo` with two URLs: reachable, refused before the hub exists.
- Card 5, `--single-repo` over an origin holding `lym-bolt` or `.lym-warp`: reachable only if this check runs before the "origin holds `lym-bolt`" route. The card lists it after that route, but the test row `SingleRepoRefusesAnOriginHoldingLymBolt` forces the order.
- Card 5, the missing `.lym-anchor` refusal, the protected or refused marker push, and teardown of the hub: reachable.
- Card 7, `refuseOffPrimeBranch` (key absent, or the prime on another branch): reachable through the three primitives the card names. The same site recurs unguarded in other records committers (finding 4).
- Card 7, `retryHeldIndexLock`: reachable through the three primitives. It is unguarded at the same other committers (finding 4).
- Card 9, step 0 already-in-progress guard: reachable.
- Card 9, `refuseSourceMarkerChange`: reachable. It also fires on a state the decision record does not mean to refuse (finding 2).
- Card 9, the records-changed refusal in `settleSourceRecords`: reachable, by an on-disk write between step 2 and step 3; the hook drives it in the test.
- Card 10, records commit around a merge, both routes: reachable through the three primitives. Unguarded at `commitAnchorMarker` and `takeoverBranch.commitEntry` (finding 4).
- Card 10, `MergeContinue` refusal on an unsettled merge, and `MidMergeRecordsUnsettled`: reachable. They also fire in two-repository hubs, where the state was never meant to occur (finding 3).
- Card 10, the abort path of `preMerge.clear` and the `ownMergeInLeftover` path of `refuseOverUnfinishedMerge` for the row's own merge-in: become unreachable for an unsettled own merge-in (finding 5).
- Card 10, the "conflicted index" refusal: its condition contradicts the card's other text (finding 7).
- Card 11, wait on an operation or a probe error, wait on `Stalled`, and completion on `Protected`: reachable. The `Protected` completion is reached for any branch (finding 1).
- Card 12, `deleteViewRefOnOrigin` with the view ref already gone, and `pushOwedCodeTip` being inert: reachable.
- Card 13, the replay stop on a local code commit, the takeover read first, and `FastForwardPair` refusal: reachable.
- Card 14, `holdPrimePushForOperator` at `PushPairAnchored`, `coalescePushSides` and `retryWeftPush`: reachable. The prime's records push recurs unguarded in `TakeOverRun` (finding 6).
- Card 14, a guard refusal recorded as `protected`: reachable.

**Findings**

1. BLOCKING, decision. Cards 4 and 11 (`pushJob.classifyFailure`, `IsPushProtected`, statuscommit's `Protected` disposition).
   - The `protected` completion is reachable on every push. That includes task branches in one repository and weft records branches in two-repository hubs, not only the prime's records push.
   - D7 and the declared `PATTERN-run-fsm` text bound the exception to "a prime records push failing as `protected`".
   - Evidence: the `IsPushOwed` class switch at `internal/fabricengine/pushanchored.go:46-58`. Card 4 classifies protected on every `pushJob`, and card 11 completes on `Protected` with no branch check.

2. MEDIUM, design. Card 9 (`refuseSourceMarkerChange`).
   - Take a task added at another anchor (card 6) and merged back into its parent. A later merge-in from the parent into that task has merge base = the task's tip (marker `frontend`) and source = the parent tip (marker `backend`).
   - The net marker change is non-empty, but no commit in base..source changes the marker. The fabric merge commit is TREESAME to its first parent on `.lym-anchor`.
   - So the guard refuses a merge-in whose source never edited the marker, which D5 does not license. No test row covers a merge-in after a back-merge.
   - Evidence: card 9 step 1 (`_lym/plan/09-merge-keeps-receiving-records.md:41`) and card 6's trailer rule (`_lym/plan/06-one-repository-add.md:35`).

3. BLOCKING, design. Card 10 (`MergeContinue`, `MidMerge`; card 9's `mergeState.RecordsSettled`).
   - `RecordsSettled` is a new bool whose zero value is false, and only the one-repository step 3 sets it (card 9).
   - Card 10's `MergeContinue` refusal and `MidMergeRecordsUnsettled` are stated without a layout scope. So every two-repository parked merge reads unsettled: `lym fabric merge --continue` refuses, and Finalize, `preMerge.clear` and loom start name only the abort. Card 9's ImpactSummary claims two-repository merges are otherwise unchanged.
   - Evidence: `internal/fabricengine/mergestate.go:47-61` (no such field today); `_lym/plan/10-records-commit-during-merge.md:59-61`.

4. MEDIUM, design. Cards 7 and 10 (records commit primitives).
   - The prime-branch refusal, the `index.lock` retry and the mid-merge route are wired only into `commitRecordsPathsHeld`, `CommitEmptyWithTrailers` and `commitWeftLocked`.
   - `commitAnchorMarker` commits the marker through `StageAndCommit` directly (`internal/fabricengine/anchormarker.go:57-82`), and so does `takeoverBranch.commitEntry` (`internal/fabricengine/takeoverwrite.go:227`).
   - So a prime off its branch takes a takeover or anchor commit on the wrong branch, and a mid-merge worktree fails it with git's partial-commit error. D4 and D7 say "Fabric refuses to commit records in the prime…" and "a records commit never skips while the worktree is mid-merge".

5. MEDIUM, decision. Card 10 (`preMerge.clear`, `loomCLI.refuseOverUnfinishedMerge`).
   - A Parent-Align or Finalize row's own merge-in that crashed before step 3 now reads `MidMergeRecordsUnsettled`, not `MidMergeParked`.
   - `ownMergeInLeftover` (`internal/loomcli/start.go:180`) and `preMerge.clear`'s own-merge-in abort (`internal/landingshed/premerge.go:50-75`) then stop matching it. Card 10 makes both name `lym fabric merge --abort` and stop.
   - D5 says the landing's pre-merge abort of the row's own parked merge-in "stay[s] as [it is]".

6. MEDIUM, design. Card 14 (`holdPrimePushForOperator` sites).
   - `TakeOverRun` writes the takeover entry on the parent's records branch, the prime's (`root`), and pushes it with `repo.PushRebaseFree()` (`internal/fabricengine/takeoverwrite.go:194`). That bypasses the guard.
   - In one repository, that push carries the operator's unpushed code commits on the prime's branch, which D7's guard forbids. No card lists `TakeOverRun`.

7. LOW, consistency. Card 10 (`commitRecordsAroundMerge` refusal list).
   - The card lists "a conflicted index" among refused states (`_lym/plan/10-records-commit-during-merge.md:53`).
   - But `MergeStateOperation` blocks a one-repository merge only when the index is conflicted on a records path (line 67), and the row `TransitionDuringACodeMergeCommitsAndPushesAndConcludeKeepsIt` expects a code-conflicted merge to commit.
   - As worded, the refusal would refuse that row.

Tool calls:
1. `mergeState` struct and `MergeContinue` in `mergestate.go` and `mergelifecycle.go`: to confirm `RecordsSettled`'s zero-value default; the manifest held neither.
2. grep for `PushRebaseFree` and `PushBranch` sites in fabricengine: to find records pushes outside card 14's guard sites.
3. `takeoverwrite.go` function headers and push loop, and `pushclassified.go` function list: to identify who owns each push site.
4. `commitAnchorMarker` body and `takeoverBranch.commitEntry`'s commit call: to check whether those records commits pass through card 7's and card 10's primitives.
