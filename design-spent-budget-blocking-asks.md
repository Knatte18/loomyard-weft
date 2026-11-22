# A spent code review budget with BLOCKING findings open asks the parent

`spent-budget-blocking-asks` · Note · bug, loom, darn, burler

## Seen

`per-task-settings-one-lookup` landed as `118bee8ad` with four BLOCKING Code-Burler findings open (F26, F2, F27, F28).
Round 2's Bouncer ruled CONTINUE on them, the budget was spent, and the round settled as an automatic accept (`cause: budget`, `automatic: true`).
`loom.code_auto_accept` is `true` in the template (`internal/loomengine/template.yaml`), and it accepts open findings, BLOCKING ones included.

## Decision

Operator, 2026-11-22: a run whose code review budget is spent with a BLOCKING finding open stops and asks its parent.
The parent accepts, or orders another round.

## Pointers

- The Bouncer already has `blocking_escalates` (`internal/shedadapters/bouncer.go`, `BlockingEscalates`): it sends a spent budget with an open BLOCKING ledger entry to the soft-cap rule instead of the auto-accept.
  No shipped code row sets it; the Code-Bouncer rows of `darn-recipe.yaml` and `tapestry-recipe.yaml` are the places to check, along with whether the soft-cap rule then escalates to the parent with cause budget.
- `escalate_decisions`, set today only in the motif and arras recipes, is the near relative for decision findings (`held-decision-lands-unruled`).
