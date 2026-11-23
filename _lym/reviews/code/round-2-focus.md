---
round: 2
focus:
  - "internal/shedadapters/bouncerdecision.go `routeDecisions` and `bouncer.go` `settleUndecided`: with escalate_decisions and blocking_escalates both set, a spent budget, a decision finding and an open non-decision BLOCKING ledger entry, does any path let the arbiter accept the round without the run's parent?"
  - "internal/loomengine/template.yaml `code_auto_accept` comment, `internal/configreg/configreg.go` and `internal/loomengine` reinterpretations: does the changed meaning of `code_auto_accept: true` ship a `configreg.Reinterpretation` per `PATTERN-config-layers`, and does `_lym/landing/summary.md` state the meaning change under Merge danger?"
  - "contracts/stencils/burler consolidation stencils (`burler-cluster-fan.md`, `burler-assist-cluster-fan.md`, `burler-assist-consolidation.md`) and `burler-review-format.md`: does a fanned hardener round keep `origin: focus` on an undriven-focus finding through consolidation?"
  - "internal/burlerengine/roundcheck.go `newFindings` and `shedadapters.BuildRoundContext`: is the `origin: focus` exemption admitted only for a harden-round segment, and does `TestCheckReviewRound` pin the refusal elsewhere?"
  - "contracts/stencils/burler/burler-step-review.md incremental-round bullet against `burler-review-format.md`: do both agree on where a new `origin: focus` finding may sit?"
  - "internal/shedadapters/bouncer_escalation_test.go rows for BlockingEscalates: does each row reach the behavior its name states, with no dropped-key rejudge preempting it?"
---

Entries name sites to settle, not expected outcomes.
