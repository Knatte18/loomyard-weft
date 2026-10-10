# Test-budget rows collide silently when two runs raise the same row

`test-budget-collision` · Note · enhancement, loom

Seen 2026-10-10: webster-audit-gaps and reed-hygiene each added one cmd/lyx test and each raised the cmd/lyx row in `cmd/lyx/testdata/test-budget.yaml` from 73 to 74.
The edits are textually identical, so the merge is clean, but the merged package overflows its budget; the failure only shows when verify runs after merging main (both runs blocked on it once) or, worse, on main after Finalize.
Idea: the budget row records a delta per branch, or the verify-after-merge failure message names the merged count to set, so the fix is mechanical and the PR-Gate reviewer need not count tests by hand.