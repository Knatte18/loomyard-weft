# Round 3 fixer-report — webster

## Fixed

- **F1** (BLOCKING, `cmd/lyx/testdata/test-budget.yaml:4`): raised the `cmd/lyx` row from 73 to 74.
  Card 4 (`4b1d3748d`) and the parent's `a5228153` each raised the row from 72 to 73 for one new top-level test, and the merge `90e02e739` kept the identical edit once while the package holds both tests.
  `lyx gate test ./cmd/lyx` passes, `TestTestBudget_PackagesStayWithinBudget` included.
  Commit `d73e6cc69`.
- **F2** (LOW, `internal/fabricengine/refscanner.go:204`): the `IsReadOnlyCommand` doc now says a glob character is rejected in every `lyx` word, quoted or not, and scopes the quoted-glob-is-text rule to `git` positionals.
  The unquoted-glob sentence it replaced said "unquoted" for `lyx` words too, which was equally incomplete, so both now match `readOnlyLyxSegment` and the `lyx fabric diff '*'` row of `TestIsReadOnlyCommand`.
  `lyx loom lint-comments` and `lyx gate test ./internal/fabricengine` pass.
  Commit `193448af6`.
- **F3** (LOW, `internal/websterengine/recoverbatch.go:42`): the `ErrRecoveryExhausted` doc states what the sentinel means, a batch that has used every counted recovery spawn, without naming a producing function.
  `lyx loom lint-comments` and `lyx gate test ./internal/websterengine` pass.
  Commit `d053507f4`.

No finding was a coverage gap, so no test was added.

## Gate repair

The round's gate refused the comment lint: three comment lines in `internal/loomcli` broke a sentence mid-line.
They came in from the parent through the merge `90e02e739` (`b3de8fd83`), not from this round's fixes.
The `removeStatusStrands` doc in `internal/loomcli/bootstrap.go` and the `TestApplyStatusStrandSurface` doc in `internal/loomcli/bootstrap_test.go` now hold each sentence on one line.
`lyx loom lint-comments --base 90e02e739^1` and `lyx gate test ./internal/loomcli` pass.
Commit `20b548818`.

## Disputed

None.

## Deferred

None.
