# Round 1 fixer-report

## Changes

- **F1** (fixed with an own change on placement):
  `internal/loomcli/redundancynote_integration_test.go` adds `TestRedundancyReportNote_ReadsTheWorktreeReport`.
  It runs `redundancyReportNote(location)()` over real hubforge pairs.
  One row writes a malformed `redundancy.json` and wants the could-not-be-read line.
  One row writes a judged report and wants the matches-HEAD line with the report's test lines.
  The review proposed folding the rows into the existing test of this note.
  That test passes fake `matchesHead` closures to `renderRedundancyReportNote` and is tier 0, so it cannot reach the closure;
  the rows need a real tier-1 pair and sit in a `_integration_test.go` file, as the package's other tier-1 tests do.
- **F2**: `internal/shedadapters/rubric_test.go` adds the row "a blank redundancy report renders none" to `TestReadRubric`, with a reader returning a single space.

Gates: `lym gate test --tier 1 ./internal/loomcli` and `lym gate test ./internal/shedadapters` pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the tier-1 test sat in the tier-0 `redundancynote_test.go`; fixed as proposed by moving it to `redundancynote_integration_test.go`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
