# Reviews ask the parent instead of settling past held decisions, open BLOCKING findings or an undriven hardener focus

`code-bouncer-asks-parent` · Task · bug, burler, loom, darn, arbiter, shed · running · PR-Review

## Goal

A code review round never settles past what its parent has to rule on.
Two faults let `per-task-settings-one-lookup` land as `118bee8ad` with its review unsettled, and a third let a hardener round approve with its focus undriven; this task fixes all three.

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

## 3. A hardener round never approves with its focus undriven

### Seen

`harden-fabric-single-repo-verbs` round 3 (sonnet, medium, 4 min) returned APPROVED with these focus entries undriven, by its own Coverage section:
the discard half of a killed teardown, the keep-local tag push, a failed step with no kill, and `lym adopt --archives` over real owner tags.
The approval settled the harden segment at once: Harden-Burler-Fix only ran its gates and the run went on to landing.
Nothing asked the arbiter or the parent, and once settled the segment cannot be reopened (`lym loom goto` refuses a review or fix row of a settled segment).
The untested cases became note `harden-fabric-undriven-cases`.

### Wanted

Operator, 2026-11-22: a hardener round never approves unless everything is in place.

- A round whose review leaves a focus entry undriven, or marks one WARN or not driven, does not settle as approved.
- The arbiter rules first: order another round over the undriven entries, or accept with a stated reason per entry.
- When the arbiter cannot rule, the run stops and asks the parent, which orders another round or accepts.

### Cause

The hardener rubric (`contracts/stencils/harden/harden-rubric-review.md`) says to walk every focus entry and record OK, WARN or FAIL,
but the shared review format (`contracts/stencils/burler/burler-review-format.md`) lets an incremental round raise only BLOCKING findings on changed hunks,
and the verdict is APPROVED whenever no finding is BLOCKING.
An undriven or WARN focus entry has no way to block, so round 3 recorded its gaps under Rejected and Coverage and could only answer APPROVED.

### Where

- The hardener recipe `contracts/recipes/hardener-recipe.yaml` and its Bouncer row (`harden_round`).
- The settle rule of a review segment for a reviewer verdict of APPROVED (`internal/burlerengine`, `internal/shedadapters/bouncer.go`).
- The arbiter's scope of halts it may settle (`internal/shedarbiter`, `arbiterstop`).

