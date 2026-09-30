# Plan-Review round 1 — fixer report

All five findings are fixed.
`lyx loom validate-plan` reports `ok: true` over the fixed plan, with only the informational `create-new-unit` findings it reported before.

## Changes

- **F1 (BLOCKING):** added `_lyx/plan/12-rebegin-keeps-start-sha.md`, a card that makes `BeginBatch` keep a prior record's non-empty `StartSHA` on a re-begin, mirroring `recover-batch`, with `integration`-tagged `beginbatch_test.go` cases.
  Added it to the overview's Card Index.
  Card 9's Phase 1 now states that the baseline's provenance rests on `recover-batch`'s inheritance and card 12, and its Intent cites D7.
  The card is appended rather than inserted before card 9 so no existing card number, commit subject, or cross-card reference changes; card 9 builds without it, and only the re-begin case depends on it.
- **F2 (LOW):** card 9 now also names the `RunResult.Outcome`/`StuckReason` doc comments and `Run`'s call-site comment about the removed done-over-a-failed-suite error.
- **F3 (LOW):** card 9's dirty-tree case now names the dirt (an untracked file and a tracked-file change identical at baseline and head), and states that a conflicting change surfaces as a restore error, pinned by card 5.
- **F4 (NIT):** card 9's return sentence limits the no-bisector notice to a regression, and the nil-bisector flaky case asserts the flaky warning is its only warning.
- **F5 (NIT):** card 10's Intent cites D11 and states why the Describe rule is needed.

## Deferred

None.
