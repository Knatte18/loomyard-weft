# Plan-Burler round 3 — fixer report

## Fixed

- **F1** (BLOCKING, design): validated.
  Card 5's replay-path dropped-key read runs on a CONTINUE or CIRCLING round on disk with no escalation record.
  `TestBouncer_CirclingGuard` (`internal/shedadapters/bouncer_circling_test.go`) replays a round-2 CIRCLING with no escalation record.
  Its rows "at the checkpoint with no recurring open key" and "a rising count over new keys" write a round-2 ledger without round 1's `alpha`, so the guard would refuse both.
  In `_lyx/plan/05-convergence-guard.md`:
  - `**Edit:**` gains `internal/shedadapters#TestBouncer_CirclingGuard`.
  - `**Intent:**` states the new row content: round 2's ledger keeps `alpha` at status `resolved` beside the row's open keys.
    `circlingEvidence` counts only a gating key open in this round and an earlier one, so both rows stay Stuck with the same pointer and next-round focus file.
  - The closing claim now covers every round-2 or later ledger a harvest or replay of any verdict reads, and holds once these rows change.
  - `**ImpactSummary:**` names the two rows beside the carry-over row.

Method check behind the claim: every `writeEvidenceLedger`, `evidenceLedgerContent`, `openGatingLedgerContent`, `circlingOpenLedger` and `bouncerLedgerContent` site in `internal/shedadapters`, `internal/shedrecipe` and `internal/loomrecipe` was compared round N against round N-1.
The only drops are the two CirclingGuard rows.
The `circlingevidence_test.go` rows call `circlingEvidence` directly and never reach the guard.

`lyx loom validate-plan` passes, with only the existing informational `create-new-unit` note on card 7.

## Disputed

None.

## Deferred

None.
