---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The bound is carried as proposed, but the rows are placed differently. Card 4 cannot build a one-repository hub, since NewOneRepoHub is created on card 5 and a Uses of it from card 4 fails uses-later-target (internal/planparser/validate.go:1062). So card 4's positive row commits .lym-anchor at the failure fixture's code prime, its negative row is the two-repository weft prime, and the task-branch row moves to card 14's TestPrimePushGuard.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: takeoverBranch.commitEntry is fixed as proposed. The proposed card inserted after card 10 to edit commitAnchorMarker would fail uses-later-target too, since checkUsesLaterTarget flags every higher-numbered card that targets an entry card 5 Uses (internal/planparser/validate.go:1062-1067). So the review's alternative is taken instead. Its two existing-worktree callers, migrateWeftMarker and recordAdoptedWeftAnchor, gain the prime-branch refusal and the retry on card 7 and a mid-merge refusal on card 10. commitAnchorMarker itself stays as card 5 uses it.
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: The commit-based rule is applied as proposed, with two changes. A source tip whose marker equals the merge base's still passes, so the card's existing way forward, a commit restoring the marker, and its row RevertedMarkerEditMerges stay true. Under the proposal alone the restoring commit is itself a newest marker commit without the trailer and would refuse. The exclusion also names IsFabricMergeCommit (two parents and the trailer) rather than the bare trailer, so a single-parent commit that carries the trailer is still judged (self-check, fix-proposal).
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed-differently
    reason: The proposal would have the overview state a two-layout wait. Instead card 11 scopes the wait to one-repository worktrees, which makes card 11 agree with the overview's promise. In a two-repository worktree MergeStateOperation blocks on any weft merge state. loom start lets a Parent-Align or Finalize run through over its own parked merge-in, which the landing aborts only after it starts (internal/loomcli/start.go:180, internal/landingshed/premerge.go:50). A wait would hold that run's first transition forever, where today's skip passes it (internal/loomcli/wiring.go:68-71).
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Round 1 fixer-report — plan

`lym loom validate-plan` passes after every change; its only finding is the informational `create-new-unit` on card 16.

## Changes

- **F1** (fixed differently; see frontmatter).
  Card 4's `pushJob.classifyFailure` records `protected` only for a push of a one-repository prime's branch: `oneRepositoryAt(j.path)` holds and `j.path` is the main worktree.
  Every other protected refusal keeps today's class, and the card adds `oneRepositoryAt` to Uses.
  Card 11's Intent states that `Protected` reads true only for the prime.
  Card 4's rows are `AProtectedBranchRefusalIsRecordedProtectedAndIsNotOwed` (the code prime with `.lym-anchor` committed) and `AProtectedRefusalOutsideAOneRepositoryPrimeKeepsTodaysClass`.
  Card 14 gains `ProtectedRefusalOfATaskBranchKeepsTodaysClass`, with `AddPairWith` added to its Uses.
- **F2.** Cards 9 and 10 use `mergeState.RecordsUnsettled` (json `records_unsettled`).
  It is saved true before step 2 in a one-repository worktree only and cleared when step 3 completes.
  Every mention and the row `RecordsWriteBetweenStepTwoAndThreeRefuses` are renamed.
  Card 10 adds the row `TwoRepositoryParkedMergeContinues` on a `NewHub` hub, and `NewHub` to its Uses.
- **F3.** Card 10 adds `ownMergeInLeftover` to its Edit list.
  `ownMergeInLeftover` and `preMerge.clear` treat the row's own unsettled merge-in as its leftover.
  Each of `TestPreMerge_ClearsOwnParkedMergeIn` and `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` gains two rows: own and another's.
- **F4.** Card 9 adds `MergeStateForTest` and `TestMergeState_Record` to its Edit list.
  Its Intent says the mirror and its converters carry the field and that `want` sets it.
- **F5** (fixed differently; see frontmatter).
  `takeoverBranch.commitEntry` is on card 7, taking the refusal and the retry, and on card 10, taking the mid-merge route.
  `migrateWeftMarker` and `recordAdoptedWeftAnchor` are on cards 7 and 10.
- **F6.** Card 14 adds `takeoverBranch.land` to its Edit list and runs the guard there.
  It adds the row `TakeoverEntryPushOfThePrimeIsHeld`.
- **F7** (fixed differently; see frontmatter).
  Card 9's `refuseSourceMarkerChange` judges only the source's own marker commits, leaving out `IsFabricMergeCommit`, which is added to its Uses.
  The card adds the row `MergeInAfterABackMergeOfATaskAtAnotherAnchor`.
- **F8.** Card 10's refusal list reads "an index conflicted on a records path".
- **F9.** Card 11 adds `statusCommitPathspec` to its Edit list and states the doc changes for both `newCommitStatusSeam` docs, `statusCommitPathspec` and `failureDisposition`.
  After F13's scoping, those docs name the one-repository wait beside the two-repository skip.
- **F10.** Card 11 drops `TestNew_ProtectedPushCompletes`, and `TestNew_PushErrorReturnsNil` gains the row `ProtectedReadsTrue`.
  Card 16 names `TestNew_PushErrorReturnsNil`.
- **F11.** Card 7 drops the row `RecordsRepoRootIsTheOneRepository`.
- **F12.** Card 1's `TestResolveWithin_OneRepository` keeps only `AnchoredDirectory` and `SuffixedCheckoutBesideIt`.
- **F13** (fixed differently; see frontmatter).
  Card 11 is rewritten so the wait applies in a one-repository worktree and `MergeActive` keeps today's skip in a two-repository one.
  The wiring sets one or the other through `lymcwd.OneRepository`.
  `TestNew_MergeActiveSkips` and `TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip` stay, and the Delete section and the edits of tests that only renamed `MergeActive` are gone.
  `TestCommitStatusSeam_Real` keeps its skip rows and gains two one-repository wait rows.
  The overview states the scoping.
- **F14.** Card 7 states what `lym.primeBranch` lets through and that it binds only a one-repository hub's main worktree.
- **F15.** Card 11 drops `Sleep` from the gained fields and from the values set.

## Self-check

The changes for the self-check findings were made after all four forks returned.

- fix-affected-set, MEDIUM: own-leftover rule does not reach `refuseOverUnfinishedMerge`. Fixed as proposed: card 10 has it consult `ownMergeInLeftover` for `MidMergeRecordsUnsettled`.
- fix-affected-set, LOW: `failureDisposition`'s doc still describes the skip. Fixed as proposed: card 11 states that doc's change.
- fix-claims, MEDIUM: card 10 contradicts its loomcli row. Duplicate of the affected-set finding; fixed by the same change.
- fix-claims, MEDIUM: the takeover's merge route can deadlock on the locks. Fixed as proposed: card 10 adds `takeoverBranch.land`, which takes the write lock before the push lock in that state, the order `lockWeftForReplay` uses.
- fix-claims, MEDIUM: card 7's "only off that branch" bound is false under `lym.primeBranch`. Fixed as proposed: card 7 restates the bound, and card 10 makes the two marker commits refuse while a git operation is in progress.
- fix-claims, LOW: the retry's stated reason is wrong. Fixed as proposed: a held-lock failure commits nothing, so the re-run commits once.
- fix-claims, LOW: battencli's doc does not say skip-while-mid-merge. Fixed as proposed: card 11 quotes battencli's own sentence.
- fix-consistency, MEDIUM: card 7 contradicts itself on the marker branch. Duplicate of the fix-claims bound finding; fixed by the same change.
- fix-consistency, LOW: card 7 claims a clone order card 5 does not state. Fixed as proposed: the reason now rests on a freshly cloned prime with no git operation in progress.
- fix-consistency, LOW: card 10's ImpactSummary is stale. Fixed as proposed.
- fix-consistency, LOW: card 10's inserted sentence takes over the bullets' colon. Fixed as proposed: one sentence names all four committers, and the bullets hang off "The route, by the merge's state:".
- fix-consistency, LOW: the new row's fixture does not match the test header. Fixed as proposed: the header names the last row's `NewHub` hub.
- fix-consistency, LOW: card 14 does not say what a held takeover does with its entry. Fixed as proposed, merged with the fix-proposal finding below.
- fix-proposal, MEDIUM: the takeover hold leaves the entry's disposition unstated. Fixed as proposed: `land` joins `dropEntry`, and the row asserts that no entry is left.
- fix-proposal, MEDIUM: card 9 leaves out fabric merge commits by the trailer alone. Fixed as proposed: card 9 names `IsFabricMergeCommit` and Uses it.
- fix-proposal, MEDIUM: the two-layout wait can deadlock a two-repository run. Fixed differently: card 11 scopes the wait to one repository instead of stating why no transition fires; this also settles F13.

## Disputed

None.

## Deferred

None.

## Rulings

None.
