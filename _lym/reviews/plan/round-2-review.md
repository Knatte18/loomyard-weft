---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: Card 4 and card 11 bound the protected class to a one-repository prime's records push, and the fixer's placement of the task-branch row in card 14 stands.
  - id: F2
    status: resolved
    answer: Cards 9 and 10 carry RecordsUnsettled, so a two-repository or older record reads settled, and card 10's TwoRepositoryParkedMergeContinues row drives MergeContinue there.
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
    answer: The fixed-differently route is sound, since commitAnchorMarker has no other caller than clone, migrateWeftMarker and recordAdoptedWeftAnchor.
  - id: F6
    status: resolved
  - id: F7
    status: resolved
    answer: The fixer's two changes hold, since a source tip whose marker equals the merge base's still passes and a back-merge in a task at another anchor finds no marker commit to judge.
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
    answer: Scoping the wait to a one-repository worktree agrees with the overview, and the overview now states the exception.
  - id: F14
    status: resolved
  - id: F15
    status: resolved
---

Every prior finding is resolved in the changed files, and no new BLOCKING defect sits on a changed hunk.

## Focus answers

- Cards 4 and 11: `pushJob.classifyFailure` records `protected` only when `oneRepositoryAt(j.path)` holds and `j.path` is the main worktree.
  Card 4's two rows reach the one-repository prime and the two-repository weft prime, and card 14's `ProtectedRefusalOfATaskBranchKeepsTodaysClass` reaches a one-repository task branch.
  Card 11's `Protected` reads `IsPushProtected`, which is true only through that class.
- Cards 9 and 10: `RecordsUnsettled` is set only by a one-repository merge before step 2, so a two-repository record and a record written before the field read settled.
  `MergeContinue`, `MidMerge`, `MergeAbort` and the records-commit route all read the unsettled side, and `TwoRepositoryParkedMergeContinues` pins that `lym fabric merge --continue` still concludes there.
  The remaining readers of the mid-merge kind (`dirtyParentReason`, `preMerge.clear`, `refuseOverUnfinishedMerge`, `ownMergeInLeftover`) are all on card 10's Edit list.
- Card 9: `refuseSourceMarkerChange` judges source commits that change the marker against their first parent, so a task added at another anchor and merged back, then merged-in from the parent, meets no such commit and passes.
  Rows `TaskAddedAtAnotherAnchorMergesBack` and `MergeInAfterABackMergeOfATaskAtAnotherAnchor` drive both directions, and card 6 gives Add's marker commit the trailer.
- Cards 7, 10 and 14: the prime-branch refusal, the index-lock retry, the mid-merge route and the guard now reach the three primitives, `takeoverBranch.commitEntry`, `migrateWeftMarker`, `recordAdoptedWeftAnchor`, and every push entry (`PushPairAnchored`, `coalescePushSides`, `retryWeftPush`, `takeoverBranch.land`).

## Rejected

- `moveWeftContent` (`anchormove.go`) commits through a raw `git commit` and so passes none of the prime-branch refusal, the retry or the mid-merge route.
  Not raised: it is on card 8, which no round-1 fix touched, and a new finding in an incremental round must be BLOCKING on a changed hunk.
- Card 11 wires `MergeActive` nil in a one-repository worktree, and `statuscommit.New` calls `deps.MergeActive()` at its two probe sites without a nil guard; the card states a nil default for the three new fields only.
  Not raised: its row `TestNew_MergeOperationWaits` runs with `MergeActive` nil and so forces the guard.
- Card 4 says the protected refusal returns "after one attempt", while `settleRejection` runs one retry push before `rejectionRetryKind` stops it.
  Not raised: no backoff sleep runs, which is the behavior the row can assert.
- Card 9 glosses `IsFabricMergeCommit` as two parents and the trailer, while card 2 also requires the tree match.
  Not raised: step 3 leaves the marker unchanged in every fabric merge commit, so the gloss changes no verdict.
