---
dispositions:
  - id: F30
    disposition: fixed
  - id: F31
    disposition: fixed
---

# Round 3 fixer report

## Changes

- F30 (D4, D4 Bound, D4 acceptance row): a records commit that finds a fabric merge stopped before D5 step 3, and `MergeAbort` on one, now commit first.
  The commit is built from a temporary index seeded from HEAD, taking the worktree's content of every records path whose index entry equals HEAD's or that neither holds,
  and HEAD's version of every path step 2 wrote from the source (index unlike HEAD's, or unmerged).
  HEAD and `WarpStart` move onto it, and the hard reset runs to it under the same hold of the write lock.
  D4's Bound states the loss: a records write after the crash to a path the source changed.
  The acceptance row names the transition's status file surviving the abort in HEAD, the worktree and origin, and `lym fabric merge --abort` keeping an uncommitted records edit.
- F31 (D5 crash paragraph, D5 acceptance row): the "Either way … aborts any merge in progress" sentence is replaced.
  A merge verb run again refuses with today's already-in-progress guard, checked before step 1 and before its dirtiness guards commit records, and never aborts;
  its way forward names `lym fabric merge --abort`, and `--continue` for a merge past step 3.
  `--continue` on a merge stopped before step 3 refuses, with `--abort` as the way forward.
  `mergeresolve`'s abort-first stays as it is.
  The acceptance row covers the refusal over a parked merge and the `--continue` refusal.
- Support log: a Round 3 entry with both picks.

## Self-check

- fix-consistency, MEDIUM: a re-run verb's step 1 or guard records commit would abort a pre-step-3 merge before the refusal. Fixed: D5 states the guard is checked before step 1 and before the guards commit records.
- fix-consistency and fix-way-forward, LOW (duplicate): "in either" over three places in the D4 acceptance row. Fixed: "in any of them".
- fix-claims and fix-proposal, MEDIUM (duplicate): `mergeresolve` is not the only abort-first path; the landing's pre-merge step aborts the row's own parked merge-in (`internal/loomcli/landingdeps.go`). Fixed: D5 and the support log name both.
- fix-proposal, MEDIUM: Finalize's parked-merge way forward (`internal/landingshed/finalize.go` `parkedMergeWayForward`) offers `--continue`, which the new rule refuses before step 3. Fixed: D5 has fabric's mid-merge read tell the two apart and Finalize name only `--abort` for a merge stopped before step 3; the D5 acceptance row covers it.
- fix-proposal, MEDIUM: a new records file absent from HEAD and the index would be left out of the commit. Fixed: D4 takes from the worktree a records path that neither holds.
- fix-way-forward, MEDIUM: the commit-then-abort sequence did not say how it holds the write lock that `MergeAbort` itself takes, leaving a window for a records write the reset drops. Fixed: D4 has the commit, the moves and the reset run under one hold of the write lock.

## Disputed

None.

## Deferred

None.

## Rulings

None.
