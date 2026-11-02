# Fixer report — code review round 1

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  `internal/planparser/join_test.go`: two refusal rows beside "an export no card delivers", "an export its only card deletes" and "an export its only card re-signs", each expecting "no card creates its export"; the existing row gained the same expectation.
  `internal/loomshed/partcheck_test.go`: a `TestValidatePart` row "export a card only deletes" with `wantCheck` `part-export-missing`.
- **F2 (LOW), fixed as proposed, wording corrected by the self-check.**
  `internal/planparser/exportname.go`: the `CheckExportName` doc comment states that the file-unit rule is applied to every handle, as for a Create declaration, and that a handle met only as a Rename New side is judged the same way.
  The optional shared helper was not made: the review asks for it only if the two copies are to stay in step, and the two copies word their details for different readers.
  No check was dropped.
- **F3 (LOW), fixed as proposed.**
  `internal/loomshed/partcheck_test.go`: a `TestNewPartsGate` subtest that writes a file under its own worktree root, exports it, and expects a failed gate with `export-existing`.
  `internal/loomcli/validateparts_test.go`: the row `AnExportTheWorktreeHoldsFails` in the validate-parts table, expecting the `export-existing` finding.

`lym gate test ./internal/planparser ./internal/loomshed ./internal/loomcli` passed with 0 failed before the self-check; the self-check's one change afterwards is a doc comment, and `go build ./internal/planparser` passed.

Commits: one per finding, except that the F1 and F3 edits to `partcheck_test.go` share one commit.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: LOW, the F2 doc comment said an export is always a Create declaration, which the Rename New side admission in `ExportMet` contradicts; fixed with my own change to the wording (the proposal's text was kept, its justification corrected).

No fork returned no report.

## Disputed

None.

## Deferred

None.
