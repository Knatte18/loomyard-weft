# Code review asks the parent for held decisions and BLOCKING findings at a spent budget

`code-bouncer-asks-parent` · Task · bug, burler, loom, darn · draft

Depends on: [`orch-autonomy-mode`](design-orch-autonomy-mode.md)

## Goal

A code review round never settles past what its parent has to rule on.
Two faults let `per-task-settings-one-lookup` land as `118bee8ad` with its review unsettled; this task fixes both.

## 1. A held decision finding gates until the parent rules

### Seen

Its Code-Burler finding F12, `member-line-overrides-explicit-off`, was a `decision`-class finding, routed to the run's parent and marked held:
the fixer leaves it alone, and only a decision record entry settles it.
The orch had put it to the operator, whose answer was still pending.
Round 2's Bouncer verdict says: "F12 is a `decision` finding deferred to the parent and does not gate."
The round then settled on budget, and the run landed with the held finding in the "Check by hand" list and the questioned exception in the code.

### Wanted

A held decision finding gates until the parent rules on it.
No acceptance, budget or otherwise, lets a run past its review round while a held finding is unruled.
The run waits for the parent's decision record entry, and the next round carries the ruling out.

### Where

- `internal/burlerengine`: the held mark (`roundcheck.go`, `prompt.go`, `fixerreport.go`).
- The Bouncer's gating rule that leaves decision findings out.
- `escalate_decisions` (`EscalateDecisions` in `internal/shedadapters/bouncer.go`), set today only in the motif and arras recipes, escalates a round holding a decision finding to the parent.

## 2. A spent budget with BLOCKING findings open asks the parent

### Seen

The same run landed with four BLOCKING findings open (F26, F2, F27, F28).
Round 2's Bouncer ruled CONTINUE on them, the budget was spent, and the round settled as an automatic accept (`cause: budget`, `automatic: true`).
`loom.code_auto_accept` is `true` in the template (`internal/loomengine/template.yaml`) and accepts open findings, BLOCKING ones included.

### Decision

Operator, 2026-11-22: a run whose code review budget is spent with a BLOCKING finding open stops and asks its parent.
The parent accepts, or orders another round.

### Where

- `blocking_escalates` (`BlockingEscalates` in `internal/shedadapters/bouncer.go`) sends a spent budget with an open BLOCKING ledger entry to the soft-cap rule instead of the auto-accept.
  No shipped code row sets it.
- The Code-Bouncer rows of `contracts/recipes/darn-recipe.yaml` and `tapestry-recipe.yaml`, and whether the soft-cap rule then escalates to the parent with cause budget.
- The `code_auto_accept` comment in `internal/loomengine/template.yaml`.
