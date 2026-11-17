## Goal

```markdown
Two defects of `lym gate redundant`'s mutation check, found by the check of five boardengine candidates (`docs/research/speed/redundant-candidates-check-2026-11-17/report.md`, `8492b54a`).
All five candidates were false: each holds a check no covering test makes, yet the verb listed it.
Both fixes live in the mutant code of `internal/testcover`.

## No mutant changes a data literal

The operators change conditions and operators, never a string, list member or map entry.

- `TestConfigTemplate_ResolvesToDefaults` is covered by `TestFabricReplay` and kills one mutant the cover also kills.
  It alone asserts `design_prefix`, the two type descriptions and the empty `labels` map.
- `TestRunHolds` is covered by `TestComputeLayers`.
  It alone feeds `Worktree-Create`, `Beam-Up`, `Finalize`, a task or non-beam project at a hold-free producer name, and a beam with no or a done status.
  A mutant of the `holdFreeProducers` list is one only a literal mutator makes.

Fix to weigh: add literal and list-member mutants, or list a candidate only when its covering tests assert every literal in its blocks.

## No mutant deletes one operand of a boolean chain

Equal block coverage plus equal kills of negated conditions does not mean equal inputs.

- `TestRunLock_BeamKindAndBeamChange` is covered by `TestRunLock`.
  Only the candidate changes a beam project's kind or beam alone, so the `before.Kind != now.Kind` and `before.Beam != now.Beam` clauses of `checkRunLocks` are exercised by it alone.
  Negating a clause breaks every accepted beam row in the cover, so the candidate has no unique kill, yet deleting a clause fails nothing else.
- `TestRunLock_SketchExemptionIsTheWholeNetChange` is covered by `TestFabricReplay`, `TestMakeSketch` and `TestCheckRunLocks_FamilyProject`.
  Only the candidate feeds a Sketch with a changed title, a changed body or a kept run status.
- `TestReadRaddleStore` is covered by `TestSlugRule_RaddleStore`.
  Only the candidate asserts the decoded `Hash` and `Project`, the tolerated unknown key and the exact `git checkout` command.

Fix to weigh: a clause-deletion mutant for each operand of a boolean chain.

## Done when

`lym gate redundant internal/boardengine` no longer lists the five candidates above.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - blockless-file-and-multi-block-attribution-untested (class design, severity MEDIUM, open in rounds 1)
  - map-literal-string-keys-held-fixed (class design, severity MEDIUM, open in rounds 1)
  - mutant-helpers-never-switched-on-in-test (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
