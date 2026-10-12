---
round: 1
focus:
  - "Lock identity and release: check that every release and hand-off in the parentlock package and its callers (loop hooks, batten backstop, lock release verb) compares on the process, so a newer owner's record is never freed."
  - "Lock Stuck halting: check that a paused lock wait and an unreadable lock record return Stuck with OutputPointer.Halt, and that PR-Gate halts at its row instead of routing to rework."
  - "Moved-parent test: check that the guard treats a code-side tip differing from the merge record's merged_tip and not holding the task's work as moved, that a missing merge record counts as moved, and that the fetch/skip-push cases follow the plan."
  - "Reroute edge: check the on_reroute three-reroute cap in the shed engine and the recipe key, and that Publish, PR-Gate and Finalize reroute to Merge-In only through the shared guard."
  - "Recipe flip: check that Merge-In and the Merge-Review perch sit ahead of Publish in both recipes, that Publish no longer merges in, and that skip_merge_review routes Merge-In straight to Publish."
  - "Run records stay ephemeral: check that the merge record, reviewed-tip file and wait record are written only under the run's .lyx scratch directory and never committed."
  - "Told geometry: check that the lock directory is resolved only in CLI wiring and told to engines, per PATTERN-told-geometry and PATTERN-hub-containment."
  - "Test economy and coverage: for each new behavior, settle whether a test covers it, and name an existing covering test before flagging a duplicate."
---

Seed focus drawn from the plan's Shared Decisions; the review range and rubric are as given.
