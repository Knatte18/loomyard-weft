# Card 17 — shed-goto-outcome

**Edit:**
- `internal/shedengine/producer.go`
- `internal/shedengine/run.go`
- `internal/shedengine/progress.go`
- `internal/shedengine/awaiting_test.go`
- `internal/shedengine/budgetexempt_test.go`

**Create:**
- `internal/shedengine/gotobudget_test.go`

**Intent:**
Declare D11's history-only `goto` outcome and make it an episode boundary for the bounce budget, so a `goto` into a segment gives every producer sharing the target's `Segment` a fresh budget.

`producer.go`: add `Goto Outcome = "goto"` beside the three verdicts, with a doc comment saying it is history-only: written by `lyx shed goto`, never returned by a producer.
The outcome constants' comment keeps "a ShedProducer must return exactly one of" the three verdicts, and `stepLocked`'s default arm still treats a producer returning `goto` as unrecognised.

`run.go`: `episodeStuckCount(history []HistoryEntry, def ProducerDef, producers []ProducerDef) int`.
Walking backward, it stops at `def`'s own most recent `done` as today, and also at any `goto` entry whose target (the entry's `Producer`) shares `def.Segment` when that is non-empty, or is `def` itself when `def.Segment` is empty.
The producers slice resolves a target's segment; a target no longer in the list ends no episode.
Update its doc comment and `stepLocked`'s call site.

`progress.go`: `Routing.Bounces` passes `r.Producers` through the new signature.

Tests: `awaiting_test.go` and `budgetexempt_test.go` move to the new signature.
`gotobudget_test.go` (untagged) covers: a `goto` to one row of a two-row segment resets both rows' counts; a `goto` to a row of another segment resets neither; a `goto` to an unsegmented row resets only that row; a `goto` naming a row absent from the list resets nothing; and a later Stuck after the `goto` counts from one.

**ImpactSummary:** Internal signature change to episodeStuckCount with two in-package callers; budgets are unchanged for any history without a goto entry.
