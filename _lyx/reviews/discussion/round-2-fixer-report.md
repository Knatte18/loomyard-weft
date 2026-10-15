# Fixer report — discussion, round 2

## Changes

- F1, fixed as proposed: D6's Rationale now says D2 replaces the plain path "for a card that meets D2's condition", pointing at D2 instead of restating it.
- F2, fixed as proposed: the Constraints validation-table bullet now cites CLAUDE.md's docs-land-in-the-same-commit rule for the `edit-test-file` row and D5's way forward.
  The `PATTERN-test-economy` bullet keeps only its extend-an-existing-table clause, and the `cmd/lyx/testdata/test-budget.yaml` clause moved to its own bullet attributed to CLAUDE.md's Test budget rule.
  Premise checked: `pattern/` mentions the budget file only in `PATTERN-test-speed.md`, never in `PATTERN-test-economy`.
- F3, fixed as proposed: Acceptance criteria gain a bullet after the `loom-plan-spec.md` one, requiring that neither the spec's card path resolution section nor `internal/planparser/doc.go` limits `Glyph.UnitPath` to the disk-existence checks, and that each names it as the sole glyph-to-path call without listing its readers.
- F4, fixed as proposed: D4 now adds that the sentence's member-glyph skip is stated of the disk-existence checks by name rather than of every `Glyph.UnitPath` reader.
  Premise checked against `contracts/specs/loom-plan-spec.md:171`, whose "skipped by both" depends on the limit D4 drops.

## Disputed

None.

## Deferred

None.
