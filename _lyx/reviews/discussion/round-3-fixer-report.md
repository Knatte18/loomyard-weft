# Fixer report — discussion round 3

## Changes

All changes are in `_lyx/discussion/decision-record.md`.

- F1 (fixed as proposed): D2's other-reasons paragraph now orders its steps per reason.
  For `branch not synced to upstream` the dirt step comes first, then the sync step.
  For `checkout is not on a branch` the checkout step comes first, citing `detachedHeadReason`, and the dirt step that follows commits on the parent branch.
  The single combined-case criterion is now two: one per reason, each pinning its order.
- F2 (fixed as proposed): the merge-in-progress arm names the parent's code checkout path from the guard error's checkout field, and its plain-git clause names `<checkout>`.
  The third acceptance criterion asserts that the reason holds the checkout path.
- F3 (fixed as proposed): the citation of Merge-In's wording now applies only to the sync step.
  The detached checkout step is stated as new text written for this reason, since `syncParent` words no detached state.
- F4 (fixed as proposed): "the plan writes the spec and `PATTERN` edit" now reads "the plan writes the spec edit".
- F5 (fixed differently): the fallback now covers a read that reports no parked merge, and the third criterion has a row for it.
  The proposal kept "the same way forward" for that read, and I did not make that change.
  With no merge record, `MergeContinue` and `MergeAbort` refuse with `*ErrNoMergeInProgress` (`internal/fabricengine/mergelifecycle.go`), so that way forward would name verbs that cannot run.
  A no-parked-merge read instead gets a way forward of only `lyx loom goto --to Finalize` and `lyx loom resume`, and D4's guard then refuses with the dirt-only form while dirt remains.
  An unreadable record keeps the merge way forward.

## Self-check

- fix-consistency, LOW: it was unclear which wording the dirt step for `branch not synced to upstream` uses. Fixed as proposed: that step now names the dirt-only wording, and the commit-on-the-branch wording is tied to the detached case.
- fix-way-forward, MEDIUM: under a no-parked-merge read the kept way forward names `--continue` and `--abort`, which refuse. Merged with the fix-proposal MEDIUM. Fixed with my own change, as described under F5 above.
- fix-way-forward, LOW: setting changes aside before the checkout left them unrestored. Merged with the fix-proposal LOW. Fixed as proposed: the checkout step now restores them on the branch afterwards.
- fix-claims, LOW: the rule for both reasons together describes a pair the guard never raises, because `upstreamSHAAt` reports a detached HEAD as having no upstream. Fixed as proposed: the sentence now says the two reasons never come together and cites that code.
- fix-claims, LOW: the sync step cited the ahead state and a "with its upstream" wording that is not `syncParent`'s. Fixed as proposed: it now cites `ParentDiverged` and its wording ("push or reconcile the parent branch in its own pair"), and notes that the guard raises the reason only on a genuine divergence.
- fix-proposal, MEDIUM: duplicate of the fix-way-forward MEDIUM; see above.
- fix-proposal, LOW: duplicate of the fix-way-forward LOW; see above.

## Disputed

None.

## Deferred

None.
