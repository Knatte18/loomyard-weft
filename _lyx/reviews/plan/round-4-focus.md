---
round: 4
focus:
  - "Card 5 (`_lyx/plan/05-landing-pre-merge-clear.md`): when `Deps.StopConflictSession` returns an error with an empty guid (for example a failed reed strand-table read in `conflictSessionStopper`), what does `preMerge.clear` return, and does any Stuck reason or way forward name an empty guid? Settle whether the planned `premerge_test.go` table has a row reaching that exit."
  - "Card 6 (`_lyx/plan/06-resume-start-own-leftover.md`): in `refuseOverUnfinishedMerge`, does the parent-branch read happen only for a parked `merge-in` state at Publish or Finalize, and does every other parked state keep today's `merge_in_progress` refusal without the read? Settle whether the planned probe-error row sets the producer that reaches the read."
---

Focus targets the two cards touched by round 3's fixes, since new text there has not been seen by a fresh reviewer.
