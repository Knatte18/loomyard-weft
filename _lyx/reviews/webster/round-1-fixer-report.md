# Webster round 1 — fixer report

## Fixed

### F1 — partitioned-member behavior in four `answerSymbols` consumers is uncovered

Validated: the card 12 rows reach a partitioned member only as a Delete target beside a whole-file Edit, and as a member beside its file for `redundant-file-target`.
No test reached a partitioned member as an Edit member span, a caller-coverage subject, a covering target or an interface method.

Commit `2e3887f64` extends existing tables and adds no top-level test, so the `planglyph` budget row is unchanged:

- `TestLaterDeleteReferences` (`deleteorder_test.go`): a later card edits `sub#Each`, declared in a `//go:build linux` file that calls `Foo` and a `//go:build !linux` file that does not, and the deletion of `sub#Foo` raises `delete-before-reference` at `sub/e.go:5` (`buildEditRegions`).
- `TestGlyphChain_CallerCoverage` (`glyphchain_test.go`), three rows over partitioned fixtures:
  - deleting `parted#Gone`, declared once per constraint set, reports its caller `callers/useparted.go` (`coverageSubjects`);
  - deleting `callees#Target` with the partitioned `callers#PartUse` on the card covers the caller inside its linux declaration (`coveredByTarget`);
  - a later card editing `callers#PartUse` leaves that caller to `delete-before-reference` (`buildEditRegions` from the caller-coverage walk).
- `TestGlyphChain_ResignScenarios` (`glyphchain_test.go`): the table gains `files`, `planGate` and `dispatch` columns, and a row re-signing `parted#Measurer.Measure`, an interface method declared once per constraint set, is `resign-interface-method` at the plan gate and clean at dispatch.

Verified by mutation: with `false` in place of the `partitioned[...]` argument at the `answerSymbols` calls in `buildEditRegions`, `coverageSubjects`, `coveredByTarget` and the `resign-interface-method` arm, each of the five new rows fails; with the code restored, `lyx gate test ./internal/planglyph` is green and `lyx loom lint-comments` passes.

## Disputed

None.

## Deferred

None.
