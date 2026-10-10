---
round: 3
focus:
  - "In `_lyx/plan/05-landing-pre-merge-clear.md`, check whether the planned `premerge_test.go` table has a row where the conflict-session stop succeeds and `AbortMerge` returns an error, so that the Stuck reason the card adds for an unabortable own leftover is reached by a test (`PATTERN-refusal-way-forward`), and that the row asserts no merge-in follows."
  - "In `_lyx/plan/06-resume-start-own-leftover.md`, check whether the narrowed `refuseOverUnfinishedMerge` guard's new error exit, when the `recordedParentBranch` seam fails over a parked `merge-in` state, is reached by a planned test row, given that `internal/loomcli/start_midmerge_test.go` already tests the sibling probe-error refusal."
  - "Across cards 5 and 6, check that any row added to cover those exits adds no target and no new top-level test the plan's `cmd/lyx/testdata/test-budget.yaml` row cannot absorb, and that the `## verify:` section still runs every package a card targets."
---

The focus targets the test coverage of the error exits that the cards add, and the cards' stated bounds on what they discard.
