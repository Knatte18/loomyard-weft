---
round: 2
focus:
  - "Card 16 (status-commit-holds-archive) against card 5 step 4, which writes `record.json`'s `class` before PR-Rework's round commit, and against `prRework.Call` in `internal/loomshed/prrework.go`, which takes `finish(true)` (clear the rejection, Done, no round commit) when `committedRejections` finds a classed record for the pending rejection at HEAD. Shed persists, and so runs `statusCommitPathspec` in `internal/loomcli/wiring.go`, on a Failed exit (a `Commit` seam error) and on the `running` write a resume makes. Settle whether any persist between the class write and the round commit can stage the classed `record.json` and the review-run deletions while the `_lyx/plan/` deletions, the new plan and the webster deletions stay uncommitted, and whether card 16's predicate and tests, and card 5's crash-resume tests, cover that window."
  - "The overview's `## verify:` section's `go test -tags integration` package list against every package a card targets, including `internal/planglyph` (card 7), which carries `//go:build integration` test files. Settle whether every targeted package's hermetic tagged tests run."
---

Both entries come from the round-1 judge pass.
The first is the remaining window of the status-commit split that card 16 was added to close; the second is a verify-coverage gap.
