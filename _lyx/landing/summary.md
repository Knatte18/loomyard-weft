# Plan gates refuse a plain _test.go path under Edit

A plan card no longer lists a plain `_test.go` path under `**Edit:**`.
Tests a card adds to an existing package are `**Create:**` declarations with `plan:` handles, and tests it changes are member glyphs under `**Edit:**`.
This removes the last glyph exception for test files: a test member follows the same glyph rules as a production member.

## What changes

- New plan check `edit-test-file` in `internal/planparser`: a file self glyph in a card's `**Edit:**` group whose path ends in `_test.go` is one blocking finding, attributed to the card.
  The detail names the way forward: member glyphs for tests the card changes, `**Create:**` `plan:` handles on the package directory (or `<dir>_test` for an external test package) for new ones, and a `**Create:**` path for a new test file.
- The check runs at the format and rework plan gates only, appended by the plan index; dispatch validation never reports it.
  At the format gate, cards in the done set are skipped, because `done-card-edited` already bars editing them.
- The check reads plan text alone, so it fires whether or not the file exists.
  It is skipped under `language: none`, and exempts paths with a segment the go tool ignores (`testdata`, `vendor`, a name starting with `.` or `_`).
- `caller-uncovered` and `redundant-file-target` no longer offer a `_test.go` path as a way forward; when the file is a test file they name only the member glyph.
- `contracts/stencils/loom/loom-template-plan.md` drops the test-file exception from the `**Edit:**` rules, removes the skeleton's `_test.go` bullet, and adds a rule: a card whose new tests reuse an existing test's setup, helpers or fakes names those members under `**Uses:**`.
  The kick-start pack then resolves them, so the fork sees their file and places the new test beside them.
- The Plan-Review rubric loses its sentence asking reviewers to flag an Edit card that lists only a test-file path, and its claim in `contracts/stencils/claims_test.go`.
- `contracts/specs/loom-plan-spec.md` gains the `edit-test-file` row and names the check in the validation preamble; its examples no longer list a `_test.go` file under `**Edit:**`.
  The card path resolution section and `internal/planparser/doc.go` no longer limit `Glyph.UnitPath` to the two disk-existence checks; they name it as the sole glyph-to-path call without listing its readers.

## Design decisions

- The plan does not pin the file a new test goes in.
  A `plan:` handle's unit is a package directory, never a `.go` file, so the fork picks the `_test.go` file in that package, as it does for any production member.
- The check has its own `refGate` and kind-policy ledger entry in `internal/planparser/shape.go`, and reads the path through `Glyph.UnitPath` (`PATTERN-ref-shape-registry`, `PATTERN-glyph-conversion-chokepoint`).
- Webster's fork stencils and kick-start pack code are unchanged: the pack already lists a card's handles, and `ScopeGuard` counts a test file edited in a handle's package as covered.
- `**Uses:**`, `**Delete:**`, `**Rename:**`, `**Move:**`, `**Prosa:**` and `**Custom:**` still accept a `_test.go` path, and non-test `.go` paths under `**Edit:**` are not gated.
- A plan an earlier binary admitted can now be refused, but only at a plan gate whose writer can fix it.
- Plan fixtures in `internal/planglyph` and `internal/loomcli` that listed a `_test.go` file under `**Edit:**` were updated, and `cmd/lyx/testdata/test-budget.yaml` was adjusted for the new tests.

## To check by hand

- Stencil wording (`stencil-caller-rule-offers-test-file`, medium, open): the plan stencil's caller rule may still offer a test file as a way forward; read the `**Edit:**` and caller rules in `loom-template-plan.md` for any remaining path to a test file.
- Plan-review rubric (`rubric-resign-arrow-bullet-merged`, `contracts/stencils/loom/loom-rubric-plan-review.md`): a bullet with the re-sign arrow was merged into its neighbour in one revision and restored in a later fix; confirm the bullet stands on its own line.
- Open risk: a fork may put a new test in a new file of the package instead of beside its neighbours.
  The `**Uses:**` rule mitigates this only where the plan writer follows it.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run by this description's author; run both before pushing.
