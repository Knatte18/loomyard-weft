---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/04-protected-push-class.md:40
    summary: The protected completion is not bounded to the prime's records push; card 4 classifies protected on every push and card 11 completes any protected-only failure, so a task branch or a two-repository weft branch refused as protected lets the run act with its records unpushed, against D7 and the declared PATTERN-run-fsm text.
    fix: "In card 4's Intent, make pushJob.classifyFailure record PushFailureProtected only for a records push of a one-repository prime's branch (oneRepositoryAt of the pushed worktree holds and it is the repository's main worktree); every other refusal ProtectedBranchRefusal matches keeps the class it gets today, so IsPushProtected and card 11's Protected read true only for the prime. State that bound in card 4's and card 11's Intent. Add a TestPushFailureClasses row AProtectedRefusalOfATaskBranchKeepsTodaysClass (the GH006 hook on a non-prime branch: class not protected, IsPushProtected false). Card 4's existing row AProtectedBranchRefusalIsRecordedProtectedAndIsNotOwed pushes the prime of a one-repository hub."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/10-records-commit-during-merge.md:59
    summary: mergeState.RecordsSettled defaults to false and only the one-repository step 3 sets it, yet card 10's MergeContinue refusal and MidMergeRecordsUnsettled carry no layout scope, so every two-repository parked merge reads unsettled and lym fabric merge --continue refuses there.
    fix: "Invert the field in cards 9 and 10: card 9 saves mergeState.RecordsUnsettled (json records_unsettled) true with the merge record before step 2 in a one-repository worktree only, and clears it when step 3 completes; card 10 reads RecordsUnsettled true wherever it now reads RecordsSettled false (the records-commit route, MergeContinue, MidMerge, MergeAbort). A two-repository record and an older record then read settled. Rename every RecordsSettled mention in cards 9 and 10, including the RecordsWriteBetweenStepTwoAndThreeRefuses row (RecordsUnsettled true). Add a TestMergeStateActive or TestRecordsCommit_DuringMerge row, or a row in an existing two-repository MergeContinue test, asserting that a two-repository parked merge still continues."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/10-records-commit-during-merge.md:63
    summary: A Parent-Align or Finalize row's own merge-in that stopped before step 3 reads MidMergeRecordsUnsettled, which ownMergeInLeftover and preMerge.clear no longer match, so card 10 makes loom start refuse and the pre-merge clear stick where D5 keeps the row's own abort-and-redo as it is.
    fix: "Add `internal/loomcli#ownMergeInLeftover` to card 10's Edit list. In card 10's Intent, have ownMergeInLeftover and preMerge.clear treat a MidMergeRecordsUnsettled merge-in of the recorded parent branch at a landing producer as the row's own leftover, aborted and redone as a MidMergeParked one is (one-repository MergeAbort handles an unsettled merge by D4); only another party's unsettled merge names `lym fabric merge --abort`. Change the TestPreMerge_ClearsOwnParkedMergeIn row to two rows: the row's own unsettled merge-in is aborted, and another source's unsettled merge names the abort. Do the same for the TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals row: own unsettled merge-in at Finalize proceeds, another refuses naming the abort."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/09-merge-keeps-receiving-records.md:50
    summary: Card 9 adds a field to mergeState but lists neither the MergeStateForTest mirror in export_test.go nor TestMergeState_Record, whose assertNoZeroFields round-trip claims every field, so the new field goes unchecked and the external test rows that assert it have no way to read it.
    fix: "Add `internal/fabricengine#MergeStateForTest` and `internal/fabricengine_test#TestMergeState_Record` to card 9's Edit list. In card 9's Intent, the mirror gains the new field (RecordsUnsettled per F2) with toMergeState and fromMergeState carrying it, and TestMergeState_Record's want sets it non-zero."
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/07-records-commit-on-code-branch.md:47
    summary: The prime-branch refusal, the index.lock retry and the mid-merge route reach only the three commit primitives; takeoverBranch.commitEntry and commitAnchorMarker commit records through StageAndCommit directly, so a prime off its branch takes them on the wrong branch and a mid-merge worktree fails them, against D4 and D7.
    fix: "Add `internal/fabricengine#takeoverBranch.commitEntry` to the Edit lists of cards 7 and 10, and state in their Intent that it runs refuseOffPrimeBranch, retryHeldIndexLock and the commitRecordsAroundMerge route like the three primitives. The same fix for `internal/fabricengine#commitAnchorMarker` is withheld: listing it on card 7 or card 10 fails uses-later-target, since card 5 Uses it. Carry it on a card inserted after card 10 with lym loom insert-card, or move the one-repository marker commits off commitAnchorMarker."
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/14-prime-records-push-guard.md:41
    summary: TakeOverRun commits the takeover entry on the parent's records branch, the prime's in a one-repository hub, and pushes it through takeoverBranch.land with PushRebaseFree, bypassing holdPrimePushForOperator and carrying the operator's unpushed code commits that D7's guard holds back.
    fix: "Add `internal/fabricengine#takeoverBranch.land` to card 14's Edit list, and list it in the Intent among the sites that run holdPrimePushForOperator before they push, for a one-repository prime. Add a TestPrimePushGuard row TakeoverEntryPushOfThePrimeIsHeld: TakeOverRun with an operator commit unpushed in the prime leaves origin's prime branch unmoved and an operator_unpushed record."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-merge-keeps-receiving-records.md:41
    summary: refuseSourceMarkerChange judges the marker change from the merge base, so a merge-in from the parent into a task added at another anchor, after that task was merged back once, sees a net marker change that no source commit made and refuses a merge D5 does not refuse.
    fix: "In card 9's Intent, judge only commits reachable from the source and not from the receiving HEAD that change .lym-anchor against their first parent, leaving out fabric merge commits (MergeTrailerKey), which step 3 settles. Refuse only when the newest such commit lacks AnchorChangeTrailerKey, and pass when none exists. Add a TestMerge_OneRepository row MergeInAfterABackMergeOfATaskAtAnotherAnchor: a task added with --anchor, merged into its parent, then merged-in from the parent, passes."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/10-records-commit-during-merge.md:53
    summary: Card 10's refusal list names a conflicted index, while its MergeStateOperation text blocks a one-repository merge only when the index is conflicted on a records path, and its row TransitionDuringACodeMergeCommitsAndPushesAndConcludeKeepsIt expects a code-conflicted merge to commit.
    fix: "In card 10 line 53, replace a conflicted index with an index conflicted on a records path, matching line 67."
  - id: F9
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/11-transition-seam-waits.md:54
    summary: Card 11 turns the skip into a wait but states the doc change only for loomcli's commitStatusDeps; the docs of loomcli's and battencli's newCommitStatusSeam and of loomcli's statusCommitPathspec still describe the mid-merge skip.
    fix: "Add `internal/loomcli#statusCommitPathspec` to card 11's Edit list. Extend card 11 line 54 to state that loomcli's and battencli's newCommitStatusSeam docs name the wait where they name skip-while-mid-merge, and that statusCommitPathspec's doc drops the clause about a transition whose commit was skipped mid-merge."
  - id: F10
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/11-transition-seam-waits.md:64
    summary: TestNew_ProtectedPushCompletes's row ProtectedNilReadsNone repeats TestNew_PushErrorReturnsNil, which already runs with Protected unset and asserts that a push error that is not owed returns. Its row ProtectedOnlyCompletes is one more push-failure disposition, so it belongs as a row of that test.
    fix: "Drop the Create target `plan:internal/statuscommit#TestNew_ProtectedPushCompletes` from card 11 and its Tests bullet. Add a row to TestNew_PushErrorReturnsNil, already on card 11's Edit list, in which Protected reads the error true: the seam returns nil, with no retry and no sleep. In card 16 line 37, name TestNew_PushErrorReturnsNil where it names TestNew_ProtectedPushCompletes."
  - id: F11
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/07-records-commit-on-code-branch.md:56
    summary: TestRecordsCommit_OneRepository's row RecordsRepoRootIsTheOneRepository repeats card 3's TestRecordsGeometry_OneRepository, which asserts RecordsRepoRoot on every row.
    fix: "Cut the row RecordsRepoRootIsTheOneRepository from card 7's TestRecordsCommit_OneRepository."
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/01-lymcwd-one-repository.md:44
    summary: TestResolveWithin_OneRepository's rows BelowTheAnchoredDirectory, WorktreeRootAboveTheAnchor and SiblingOfTheAnchor repeat the cwd gate TestResolveWithin pins through its AnchoredDirectory-family rows, and card 1's TestReadRecordedAnchor_MarkerSemantics rows pin the root-marker read.
    fix: "In card 1, keep TestResolveWithin_OneRepository's rows AnchoredDirectory, which pins the root-marker read through the resolver, and SuffixedCheckoutBesideIt, which pins the ownerOfRecordsSibling change; cut BelowTheAnchoredDirectory, WorktreeRootAboveTheAnchor and SiblingOfTheAnchor."
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/00-overview.md:13
    summary: The overview says two-repository hubs keep today's behavior throughout, but card 11 makes the transition seam wait instead of skip in both layouts, since MergeStateOperation blocks on any weft merge state in a two-repository worktree.
    fix: "In 00-overview.md line 13, add after Two-repository hubs keep today's behavior throughout: except the transition seam, which waits on a blocking git operation in both layouts (card 11), so no transition is passed with its records uncommitted. State the same two-repository change and its reason in card 11's Intent."
  - id: F14
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/07-records-commit-on-code-branch.md:41
    summary: Card 7 introduces lym.primeBranch, which picks the one branch the prime commits records on, but its Intent states only the stall its refusal causes, not what the key can now let through and what bounds it.
    fix: "Add to card 7's Intent, after line 49: the key lets records commits land on whichever branch it names while the prime has that branch checked out, so an operator who sets it to another branch moves the prime's records there; the key binds only the main worktree of a one-repository hub, never a task worktree or a two-repository records worktree, and every records push it leads to still passes card 14's guard."
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/11-transition-seam-waits.md:51
    summary: Card 11 says loomcli's and battencli's commitStatusDeps gain a Sleep field, but both already carry Sleep set to time.Sleep.
    fix: "In card 11 line 51, drop Sleep from the gained fields and from the values set, leaving MergeOperation, Stalled and Protected."
---

### [BLOCKING] The protected completion reaches every push, not only the prime's records push

**Location:** `_lym/plan/04-protected-push-class.md:40`

**Issue:** D7 and the declared `PATTERN-run-fsm` text allow one exception to "a transition commits and pushes before the run acts": "a prime records push failing as `protected`".
Card 4's `pushJob.classifyFailure` records `PushFailureProtected` for any refusal `ProtectedBranchRefusal` matches, with no branch or layout check.
Card 11 then completes on `Protected` (`IsPushProtected`), again with no branch check.
So a task branch in a one-repository hub, or a weft records branch in a two-repository hub, refused under a wildcard protection rule completes its transition and the run acts with its records unpushed.
Today such a push is not owed (`internal/fabricengine/pushanchored.go:46-58`) and halts the run.
This breaks a declared PATTERN entry, so it is BLOCKING.

### [BLOCKING] RecordsSettled's zero value marks every two-repository parked merge as unsettled

**Location:** `_lym/plan/10-records-commit-during-merge.md:59`

**Issue:** Card 9 adds `mergeState.RecordsSettled`, saved true only by the one-repository step 3.
Card 9's own ImpactSummary says an older record reads the field as unset.
Card 10's `MergeContinue` refusal and `MidMergeRecordsUnsettled` (lines 59 and 61) are not scoped to one repository.
A two-repository merge never runs step 3, so every two-repository parked merge reads unsettled.
`lym fabric merge --continue` then refuses there, and Finalize, `preMerge.clear` and loom start name only the abort.
Both the overview and card 9's ImpactSummary say two-repository merges are unchanged.

### [MEDIUM] A landing row's own unsettled merge-in is no longer its own leftover

**Location:** `_lym/plan/10-records-commit-during-merge.md:63`

**Issue:** `ownMergeInLeftover` (`internal/loomcli/start.go:180`) and `preMerge.clear` (`internal/landingshed/premerge.go:50`) recognise the row's own merge-in only as `MidMergeParked`.
Card 10 reports a merge stopped before step 3 as the new kind, and routes both readers to a refusal naming `lym fabric merge --abort`.
So Parent-Align or Finalize, resumed after its own merge-in crashed before step 3, refuses to start and its pre-merge step sticks.
D5 says "the landing's pre-merge abort of the row's own parked merge-in" stays as it is, and card 10's one-repository `MergeAbort` handles an unsettled merge safely.
`affected-set` and `guard-reach` both report this defect.

### [MEDIUM] Card 9 misses the merge-state test mirror

**Location:** `_lym/plan/09-merge-keeps-receiving-records.md:50`

**Issue:** `MergeStateForTest` (`internal/fabricengine/export_test.go:526-543`) mirrors `mergeState` field by field.
`TestMergeState_Record` (`internal/fabricengine/mergestate_integration_test.go:51-97`) carries `//lym:seam internal/fabricengine#MergeStateForTest` and runs `assertNoZeroFields` so that the round-trip covers every field.
Card 9 adds a field without listing either, so the record's new field round-trips unchecked.
The rows in cards 9 and 10 that assert it have no exported way to read it.

### [MEDIUM] Records commits outside the three primitives skip the prime refusal, the retry and the mid-merge route

**Location:** `_lym/plan/07-records-commit-on-code-branch.md:47`

**Issue:** `takeoverBranch.commitEntry` (`internal/fabricengine/takeoverwrite.go:227`) and `commitAnchorMarker` (`internal/fabricengine/anchormarker.go:79`) commit records with `StageAndCommit` directly.
Cards 7 and 10 wire the D4 and D7 behaviour only into `commitRecordsPathsHeld`, `CommitEmptyWithTrailers` and `commitWeftLocked`.
In a one-repository prime off its recorded branch, a takeover entry then lands on the operator's other branch, and a takeover during a merge fails with git's partial-commit error.
Adding `commitAnchorMarker` to card 7 or card 10 fails the plan gate's `uses-later-target`, since card 5 Uses it, so that part of the fix is withheld and names that check.

### [MEDIUM] The takeover push of the prime bypasses the operator guard

**Location:** `_lym/plan/14-prime-records-push-guard.md:41`

**Issue:** `TakeOverRun` writes the takeover entry on the parent's records branch and pushes it through `takeoverBranch.land` with `repo.PushRebaseFree()` (`internal/fabricengine/takeoverwrite.go:168-194`).
With no parent given, the parent is the prime.
In one repository that branch is the operator's code branch, so the push carries any unpushed operator commits, which D7 says a records push of the prime never does.
Card 14 guards `PushPairAnchored`, `coalescePushSides` and `retryWeftPush` only.

### [MEDIUM] The marker-change refusal fires after a back-merge of a task at another anchor

**Location:** `_lym/plan/09-merge-keeps-receiving-records.md:41`

**Issue:** `refuseSourceMarkerChange` compares the marker at the source tip against the merge base.
Take a task added with `--anchor`, which card 6 allows, merged once into its parent and then merged-in from the parent.
The merge base is the task's tip (marker `frontend`) and the source is the parent's merge commit (marker `backend`, kept by step 3).
The net change is non-empty, yet no source commit edited the marker, so the merge-in refuses.
D5 refuses only a source that changed the marker other than through fabric's anchor change.

### [LOW] Card 10's refusal list contradicts its own operation rule

**Location:** `_lym/plan/10-records-commit-during-merge.md:53`

**Issue:** Line 53 refuses on "a conflicted index".
Line 67 blocks a one-repository merge only when the index is conflicted on a records path.
The row `TransitionDuringACodeMergeCommitsAndPushesAndConcludeKeepsIt` commits during a code conflict.

### [LOW] Skip wording left in three seam docs

**Location:** `_lym/plan/11-transition-seam-waits.md:54`

**Issue:** These doc comments still describe the skip card 11 removes:
- loomcli's `newCommitStatusSeam` names "skip-while-mid-merge";
- battencli's `newCommitStatusSeam` says "It skips while a merge is in progress";
- loomcli's `statusCommitPathspec` says "a transition whose commit was skipped mid-merge is caught up by the next one".

Card 11 lists the first two but states no doc change for them, and lists `statusCommitPathspec` nowhere.

### [LOW] TestNew_ProtectedPushCompletes repeats TestNew_PushErrorReturnsNil

**Location:** `_lym/plan/11-transition-seam-waits.md:64`

**Issue:** `TestNew_PushErrorReturnsNil` (`internal/statuscommit/statuscommit_test.go:212`) drives a push error that is not owed with `Protected` unset and asserts the error returns.
That is the row `ProtectedNilReadsNone`.
The protected completion is one more push-failure disposition of the same table, so it belongs there as a row, not in a new function.

### [LOW] RecordsRepoRoot asserted twice

**Location:** `_lym/plan/07-records-commit-on-code-branch.md:56`

**Issue:** Card 3's `TestRecordsGeometry_OneRepository` asserts `RecordsRepoRoot` on every one of its rows, `OneRepository` included.
Card 7's row `RecordsRepoRootIsTheOneRepository` pins nothing more.

### [LOW] TestResolveWithin_OneRepository re-pins the cwd gate

**Location:** `_lym/plan/01-lymcwd-one-repository.md:44`

**Issue:** `TestResolveWithin` (`internal/lymcwd/lymcwd_test.go:342`) pins `checkCwdWithinAnchor`'s accept and refuse cases.
`TestReadRecordedAnchor_MarkerSemantics` gains the root-marker rows.
Only `AnchoredDirectory` (the root-marker read end to end) and `SuffixedCheckoutBesideIt` (the `ownerOfRecordsSibling` change) pin behaviour no other test covers.

### [LOW] The overview's two-repository promise contradicts card 11

**Location:** `_lym/plan/00-overview.md:13`

**Issue:** Card 11's ImpactSummary says `New` "waits where it skipped in both layouts".
Card 10's `MergeStateOperation` blocks on any weft merge state in a two-repository worktree.
So two-repository runs now wait at a weft merge instead of skipping it.
The overview says two-repository hubs keep today's behavior throughout.

### [LOW] lym.primeBranch carries no stated bound

**Location:** `_lym/plan/07-records-commit-on-code-branch.md:41`

**Issue:** The key decides which branch the prime commits records on, and the refusal's way forward tells the operator to set it.
Card 7 states the stall while the prime is off the branch.
It does not state what the key lets through, or that it binds only the main worktree of a one-repository hub.

### [NIT] Sleep is not a new field

**Location:** `_lym/plan/11-transition-seam-waits.md:51`

**Issue:** `commitStatusDeps` in `internal/loomcli/wiring.go` and in `internal/battencli/commitstatus.go` already carry `Sleep`, and both builders already set it to `time.Sleep`.

## Rejected

- The overlap of `TestNewOneRepoHub/CloneSecondTakesTheSameLayout` with `TestCloneAndWire_OneRepository/OneURLOverLymBolt`: the hubforge row pins `CloneSecond`'s own arguments, an empty `RecordsBare` and `singleRepo` false, which the clone test does not reach.
- D3's hub workspace bullet carried by no card: `hubWorkspaceFolders` lists folders by directory presence (`internal/fabricengine/hubworkspace.go:317`), and a one-repository hub has no sibling directory to list, so it needs no change.
- The `guard-reach` note on card 5's clone-route order: the row `SingleRepoRefusesAnOriginHoldingLymBolt` forces the `--single-repo` refusal to run before the `lym-bolt` route, so the refusal is reachable as planned.
