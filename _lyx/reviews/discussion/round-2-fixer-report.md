# Round 2 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

## Fixed

- F1: D11's Decision now states that a file-level comment excuses only the tests touching the state it names, and that a test touching none calls `t.Parallel`; the Rationale keeps only the reason.
- F2: Acceptance criteria now list each removed phrase as its file carries it, per file, including the spec's "independently reviewable and testable unit", "independent meaning or testability" and the "its-own-new-test-file shape", and the plan stencil's "so it is testable".
- F3: D5's Webster-Review bullet lists each case with what its finding names.
- F4: D1, D10, D11 and D12 open with "Landed in the Discussion commit (D13)", and D13 says the driver committed it.
- F5: D1's last bullet, which the background file lacks, is dropped; Scope's Out list keeps the exclusion.

## Deferred

None.
