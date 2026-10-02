# Fixer report — plan, round 1

All seven findings fixed.
`lyx loom validate-plan` still exits 0, with only the two informational `create-new-unit` findings, and `approved:` stays `false`.

## Changed

- **F1** (`01-shuttle-gate-list.md`): the interim `resolveGateSpec` maps an absent, zero or negative `gate_attempts` to `Attempts` 3, today's default reading.
  The card now names the one changed assertion (`AbsentGateAttemptsYieldsPackageDefault`, 0 to 3) and the ungated subtests' switch to an empty-list check.
  The Intent's "every existing test keeps its assertions" line now carves out that assertion.
- **F2** (`01-shuttle-gate-list.md`): the count slices are sized from `len(run.gate)` on first use, so both `Run` literals and bare test literals need no new field.
  `internal/shuttleengine/attach.go` is added under Uses.
- **F3** (`02-recipe-gates-key.md`): `internal/loomrecipe/resume_test.go` and `internal/loomcli/smoke_gate_test.go` are added to Edit, with a sentence on their `gate: discussion` comments.
  The `gatequiescence_test.go` sentence now covers its doc comments and failure messages.
- **F4** (`01-shuttle-gate-list.md`): the doc-comment sentence now covers every comment in the edited files that names `gate.Gate`, `opts.Gate.Gate` or a nil `Gate`, naming the `burlerengine/engine.go` and `shedadapters/singlellm.go` sites.
- **F5** (both cards): Card 1's ImpactSummary names the eight migrated packages and the pass-through callers.
  Card 2's says the old keys are read only by `shedrecipe` and `gatequiescence_test.go`.
- **F6** (`02-recipe-gates-key.md`): `resolveGateSpec` wraps helper errors for a `gates` element with the entry and element index.
  The leftover-key rejection names the key, and `shedbuild`'s per-row wrap names the row; the per-entry leftover-key tests assert that the error names the old key.
- **F7** (`00-overview.md`): one Shared Decisions line records that Told-Geometry, Transient Stop and Refusal Way-Forward hold without a change.

## Deferred

None.
