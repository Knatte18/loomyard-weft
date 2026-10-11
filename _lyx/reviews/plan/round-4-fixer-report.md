# Plan-Review round 4 — fixer report

## Fixed

- **F1 (MEDIUM, design):** the premise holds: `RecordCirclingDecision` (`internal/shedadapters/circling.go:46-66`) records a decision over a legacy CIRCLING round with no escalation record.
  The Shared Decision `replay-guard-skips-an-escalated-round` in `00-overview.md` now exempts a round carrying either an escalation record or a circling decision file, and its rationale names the legacy case.
  Card 5's replay-path sentence now skips the guard when `readEscalation` reports a record or `readCirclingDecision` reports a decision file.
  `TestBouncer_JudgeCall_RefusedJudgment` gains the row: a replay of a round-2 CIRCLING with a lossy ledger, no escalation record and a pending accept recorded, which settles Done.
- **F2 (LOW, consistency):** the premise holds: `carriesOver` (`internal/shedadapters/bouncercarryover.go:85-90`) admits BLOCKING as its own disjunct, and `TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` carries only `alpha` at MEDIUM.
  Card 5 adds that test to `**Edit:**`; its Intent states the new content: the test lays out its run directory through `layoutCirclingBouncer`, round 2's ledger adds `beta` open at `design`/`BLOCKING` beside `alpha`, and the expected carry-over gains `beta` with rounds `[2]` after `alpha`.
  The ledger stays lossless and the accept replay is exempt under its decision file, so the outcome is unchanged.
  The commit-test sentence no longer claims filter coverage, and the ImpactSummary names the new test edit.

## Disputed

None.

## Deferred

None.
