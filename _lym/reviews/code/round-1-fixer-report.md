# Round 1 fixer-report

The gate `lym gate test ./internal/testcover` passes.
Fixes were made in one batch and committed in two commits, not one per finding, because the findings share files and the round's rules require one edit pass.

## Changes

- **F1** fixed as proposed.
  `otherFileMiss` and `embedPatterns` take a flag; the importing package reads its `_test.go` files' `go:embed` patterns, a linked package its non-test files only.
  The embeds cache is keyed by `embedScope`.
  Row `OwnTestFileEmbedsInertFile` pins it.
- **F3** fixed as proposed: row `EditInsideMethodBody`.
- **F4** fixed with one own change.
  Rows added for each listed case, with fixture support for `committed` and `commitEdits` per row.
  A method named `max` does not give "no miss" as the review's `fix` says: an added method is a method-set change, so the row `AddedMethodNamedMax` wants `MissMethodSet`, which still fails if the predeclared check also applied to methods.
- **F5** fixed as proposed: rows for quoted, backquoted, `all:`, two-pattern, glob, malformed and unparseable-file embed directives, and an inert edit under an unrelated directory.
- **F6** fixed with an own change.
  The proposal skipped the removed-import check for a deleted file wholly, but a deleted file's blank import of a module package can drop that package's `init` from the build (fix-proposal lens).
  `compareGoSources` now returns the deleted file's imports as `removedImports`, and the selector misses unless each is standard library.
  The `a/gone.go` fixture imports `strings`, so row `DeletedFile` pins the no-miss case; row `DeletedFileBlankImportsModulePackage` pins the miss.
  `doc.go` states the rule.
- **F7** fixed with an own change.
  `go/build` finds no GOROOT in a `-trimpath` test binary, so the standard-library check stats a directory under the new `SelectorConfig.GoRoot`; an empty `GoRoot` means no import is standard library.
  The tests build a fake `GoRoot`.
  Row `AddedImportOfDotlessNonStandardPath` pins it.
- **F8** fixed as proposed: the comments in `testselect.go` and `declcompare.go` stand alone.
- `doc.go` also names the own-test-file embed rule from F1.

## Self-check

- fix-code-correctness, LOW: row `DeletedFileWithImport` duplicated `DeletedFile` — fixed as proposed, row deleted.
- fix-code-correctness, LOW: empty `GoRoot` stats a relative path — fixed as proposed, guard and field comment.
- fix-consistency, LOW: `doc.go` removed-import bullet stale after F6 — fixed as proposed, `doc.go` updated.
- fix-consistency, LOW: `doc.go` omits own test files as embed sources — fixed as proposed.
- fix-consistency, LOW: `GoRoot` not documented as required — fixed as proposed.
- fix-proposal, MEDIUM: deleted file's blank import of a module package narrows the run — fixed with an own change (see F6).
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
`_lym/discussion/decision-record.md` line 52 words D5's import rule without the deleted-file exception; the file is outside the write surface, so the parent may update it.

## Rulings

- **F2** (spawn-marked tests selected whenever a linked function changed): the ruling is to keep the code as it stands.
  The code already does, so no change was made.
