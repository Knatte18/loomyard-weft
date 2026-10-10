# PR-Gate review: reed-hygiene (PR #529)

Verdict: REJECT, one fix on the branch; the rest of the goal is done and approved.

## F1 (BLOCKING): cmd/lyx test budget overflows after the merge with main

The branch adds one top-level test to `cmd/lyx` and raises its row in `cmd/lyx/testdata/test-budget.yaml` from 73 to 74.
Main (800a0e892, webster-audit-gaps) also added one `cmd/lyx` test and raised the same row from 73 to 74.
The two edits are textually identical, so the merge is clean, but the merged package holds 75 top-level tests against a budget of 74, and `TestTestBudget_PackagesStayWithinBudget` fails on main after Finalize.

Fix: merge main into the branch, set the `cmd/lyx` row to the merged count (75), and run `lyx gate test ./cmd/lyx`.
Change nothing else.
