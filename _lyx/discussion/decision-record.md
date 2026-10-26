# Decision record — rename-target-checks

## Goal

A symbol a batch renames or removes while a later card still targets it is caught before the next `begin-batch` refuses with `plan_conflict: tree_moved`.
Three layers catch it: the plan writer declares the renames of the tests named after a renamed symbol, plan validation flags a Rename pair whose same-named tests stay under `Edit`, and record-batch hands a batch that removed a later card's target back to its fork while the fork still holds the batch's context.

## Scope

- The plan writer stencil `contracts/stencils/loom/loom-template-plan.md`: a card that renames a symbol also declares, as Rename pairs on the same card, the renames of the tests named after it.
- A new plan-format check `rename-leaves-test` in `internal/planparser`, documented in `contracts/specs/loom-plan-spec.md` and the package doc.
- A new read-only batch-end check `later-target-removed` in `internal/planglyph`, reached through a new `planindex.Delta` method.
- record-batch refuses a fork batch whose OK report left a later card's target removed, once per batch, with a new CLI flag on `lyx webster record-batch`'s envelope.
- The webster Master stencil gains the rung that sends the refusal to the batch's fork, and the implementer body says how a fork answers it and that a renamed test named by a card is a rename it reports.
- Docs: `internal/websterengine/doc.go`, `internal/planglyph/doc.go`, `internal/planparser/doc.go`, the record-batch help text.
- Out of scope: recover-batch, begin-batch's own refusal, and the plan-review rubric.

## Decisions

### Decision: the plan check is `rename-leaves-test`, Go plans only

A symbol Rename pair whose old side is a Go member glyph `unit#Name`, or `unit#T.M` for a method, names the test base `Name`, or `T_M`.
Any card's `Edit` ref in the same unit naming a function `Test<base>` or `Test<base>_<suffix>` that no Rename pair of the plan renames is one blocking finding, attributed to the renaming card and naming the card that edits the test.
The way forward is to add the test's rename as a Rename pair on the renaming card.

Rationale: the incident's tests were `TestNewBeam` and `TestBeamRouting` beside the renamed `NewBeam` and `BeamRouting`, which is Go's test naming convention.
Requiring `_` after the base keeps `TestNewBeamer` out of `NewBeam`'s set.
A C# plan names tests by other conventions, so the check is silent under `csharp` and `none`.

### Decision: the batch-end check is a read-only Delta method run before handle binding

`planindex.Delta` gains `RemovedTargets(fullPlan, completed, own, reported)`, implemented in `internal/planglyph`.
It reports every symbol the delta deleted that a pending card targets or uses, as a blocking `later-target-removed` finding per card, naming any evidence-tier rename candidates quarry offered for it.
A symbol is no finding when quarry renamed it exactly (quarry drops it from the deleted list), when the batch's report lists it under `renames`, or when one of the batch's own cards declares it as a Delete target or a Rename pair's old side.
postBatchChecks runs it right after the shared delta and before BindHandles, so a refusal leaves the plan unrewritten except for a C# re-sign rebind, which a retried record-batch already re-runs today.

Rationale: a declared delete or rename referenced later is a plan defect that plan validation and the existing later-card warning own, never the fork's.
Running before any plan rewrite keeps a retried record-batch from seeing a half-applied bind or drift repair.

### Decision: record-batch returns the batch to its fork once per batch

On an OK report from a fork batch with `later-target-removed` findings, record-batch refuses with `ErrLaterTargetRemoved`, leaving the batch non-terminal, its report in place and its bracket open, and records on the batch that it was returned.
The CLI saves the whole run state, syncs fabric and exits non-zero with `{"later_target_removed": true, "batch": "NN-<slug>"}`.
The refusal's way forward is for the fork to restore the name, or to list the rename under its report's `renames`, rewrite its report with its new head, and for Master then to call record-batch again.
A second record-batch on a batch already returned runs no such refusal: the batch records as today, with DetectDrift's later-card warning and the later card's own begin-batch refusal.

Bound: the refusal can delay a batch by at most one extra fork turn, since it fires once per batch record and never on a FAILED or NEEDS_PARENT report.
It skips nothing: the second call runs every check the batch runs today.
recover-batch never refuses on it, since a recovery strand ends with its report and holds no context to answer.

Rationale: the fork that made the rename still holds the batch's context and is the cheapest party to say whether it was meant.
The once-per-batch bound gives Master a way forward when the fork cannot be reached, such as after a resume.

### Decision: Master sends the refusal to the fork by SendMessage

The Master stencil's failure ladder gains a rung for `later_target_removed`: send the refusal's message to the batch's fork by `SendMessage`, end the turn, and on the fork's completion notification call `record-batch <NN>` again.
When the fork can no longer be reached, Master calls `record-batch <NN>` again at once.
The implementer body tells a fork receiving that message to restore the name or list the rename under `renames`, commit any restore, rewrite its report with the new head, and end.

Rationale: Master already continues forks it spawned only through their completion notification; a SendMessage to the fork keeps its context, which a re-fork would not.

## Constraints

- `PATTERN-refusal-way-forward`: the new refusal names its way forward, and a test reaches it.
- `PATTERN-tolerant-records`: the new BatchState field is additive and read tolerantly.
- `PATTERN-sole-parsers`: the plan check lives in `internal/planparser`, and the glyph reading goes through `ParseRef`.
- `PATTERN-stencil-ownership`: stencil text stays in `contracts/stencils`, read at call time.
- `PATTERN-test-speed`: each new behavior is a row or step in the existing test of its behavior, in the lowest tier that fits.

## Auto-mode assumptions

- The board entry's "the batch gets a departure" means the batch is handed back to its fork with the finding, not a `Pattern-Departure:` trailer.
- "A symbol a later card targets" covers a later card's `Uses:` refs too, matching the reference index drift detection already reads.
- The plan check attributes its finding to the renaming card, since the fix lands there.

## Open risks

- A fork renaming a symbol for a reason of its own and then reporting the rename moves the later card's ref through the reported-rename repair, which trusts the fork's new glyph; the post-repair resolve still blocks a new glyph that does not resolve.
- quarry may report a symbol deleted whose file moved between packages; the later card then sees the refusal once and the existing warning after.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - rename-leaves-test-misses-external-test-package (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- A Go plan whose Rename pair renames `X` while a card lists `TestX` or `TestX_Case` under `Edit` with no Rename pair for it draws one blocking `rename-leaves-test` finding on the renaming card; with the test's pair declared, or under `csharp`, it draws none; `TestXY` draws none.
- A method rename `T.M` matches `TestT_M`.
- record-batch on an OK fork report whose batch removed a symbol a pending card targets refuses with `ErrLaterTargetRemoved`, keeps the batch non-terminal and its report present, and names the symbol, the card and the way forward.
- A second record-batch on the same batch records it done, with the later-card warning as today.
- A removed symbol the report lists under `renames`, an exact rename, and a symbol one of the batch's own cards deletes or renames by declaration draw no refusal.
- The CLI envelope carries `later_target_removed: true` and the batch name, and the returned flag survives a state save.
- recover-batch never refuses on a removed later target.

## Notes for the plan writer

The refusal sits in `postBatchChecks` behind an input flag that only `RecordBatch` sets for an OK report.
