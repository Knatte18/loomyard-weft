# Round 1 fixer report — discussion

## Changes

- F1, fixed as proposed: D4 now has the `loom-plan-spec.md` sentence and its `internal/planparser/doc.go` twin drop the already-false two-reader limit and name `Glyph.UnitPath` as the sole glyph-to-path call without listing readers; Scope's spec bullet points at D4's rewrite.
  Premise checked: `Glyph.UnitPath` is also read in `reviewclass.go`, `redundancy.go` and `targetdirs.go`.
- F2, fixed as proposed: the Constraints bullet binds to `loom-plan-spec.md`'s validation-table convention, and D5's rationale no longer cites `PATTERN-refusal-way-forward`.
  Premise checked: `refusal-spec.md`'s loom section says `validate-*` findings are verdicts with no row there.
- F3, fixed as proposed: D3 states why the `language: none` skip exists, what it lets through and what bounds it.
- F4, fixed as proposed: D6's rationale says D2 replaces the whole-file read only for a card whose new tests reuse existing setup, pointing at Open risks.
- F5, fixed as proposed: Open risks' second bullet deleted, D6's hedge replaced, support-log Q7 marked resolved.
  Premise checked: `RefScope` passes a `plan:` handle's glyph unit through `goPackageDir`, and `ScopeGuard`'s coverage calls `RefScope`.
- F6, fixed as proposed: the stencil and spec acceptance criteria now cover the `**Create:**` test-file rule rewrite and the validation-check preamble entry.
- F7, fixed as proposed: D3's exemption and the third acceptance criterion cover every segment the go tool ignores.
  For consistency, support-log Q4's answer now says the same.

## Disputed

None.

## Deferred

None.
