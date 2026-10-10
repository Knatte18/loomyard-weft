# Fixer report: discussion round 3

All four findings were validated against the code and fixed in `_lyx/discussion/decision-record.md`; `lyx loom validate-discussion` passes.

## Fixed

- **F1 (BLOCKING, design).**
  Confirmed against `contracts/specs/refusal-spec.md`'s loom section, which holds `Webster-Burler verify gate exhausted`, `Publish: verify failed` and the gate-slot row naming only the existing sites.
  Added D13, a table of every loom-section row the change adds (the two `darn.yaml` refusals, `Darn` verify gate exhausted, Finalize parent run records commit failed) or edits (`Publish: verify failed`'s Way forward cell, the gate-slot trigger), each with its way forward.
  D3 now states that the spent-budget Stuck reason names `lyx loom resume`.
  D7 points at D13 instead of counting two refusals.
  The `PATTERN-refusal-way-forward` constraint and the Docs acceptance criterion name D13's rows, and a `Darn` criterion pins the spent-budget way forward.
- **F2 (MEDIUM, design).**
  Confirmed against `internal/landingshed/verifygate.go`'s recorded-failure reason, which names the Webster-Review round.
  D4 gains a prior-work token: a Go-rendered note that names the latest `Darn` outcome and the Publish failure record, or `(none)`.
  The stencil tells the agent to continue from the branch's commits when the token is set.
  D6 now takes Publish's whole way-forward clause from the recipe, and darn's clause names the `Darn` goto and the darn gate.
  Acceptance criteria cover the token and both Publish clauses.
  The support log's Rejected alternatives records the stencil-only option.
- **F3 (LOW, consistency).**
  Confirmed against `internal/loomcli/reject.go`'s hard-coded `PR-Rework` state check, refusal text and `Long` help.
  D1 now says `reject` takes its rework row from the recipe.
  It accepts a darn run blocked at `Darn` with a pending rejection record, refuses one blocked there with none, and names only the recipe's rows.
  D12 and the Docs criterion add `reject`'s help, and a `Darn` criterion pins the state check.
- **F4 (NIT, consistency).**
  Confirmed that `loomrecipe.Routing` reads only the review budget.
  D4 now names the verbs that build the shed (`step`, `run`, start, resume) as the ones the `verify_attempts` guard fires on.
  It also says `goto` is not refused, since darn's routing reads no `verify_attempts`, and the next step refuses instead.
  D7 says "every shed build" instead of "every arm".

## Disputed

None.

## Deferred

None.
