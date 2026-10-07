# Round 3 fixer report

All three findings are fixed, one commit each, none pushed.

## Fixed

- **F1 (LOW)** — `9e6a6ea45`: `IsReadOnlyCommand` now looks up each segment's first word in its original spelling, so a first word holding a quote or a backslash, whole or in part, is never read-only.
  The segment boundaries still come from the quote-blanked copy, which stays rune-aligned with the command; `commandWordQuoted` and its `unicode` import are gone.
  The new `TestIsReadOnlyCommand` row `cat"x" y` failed before the change and passes after it; a second row, `\cat x`, pins the backslash case the doc line now claims.
- **F2 (LOW)** — `c48181fb8`: the assertion in `TestRecoverSpawnOrAttach` also pins the HEAD-at-start wording ("when the worktree is clean and HEAD is the batch's start commit"), beside the committed-route wording and the two verb steps.
  No new test.
- **F3 (LOW)** — `3390931b6`: `readOnlyEntry` in `acceptbatch_test.go` builds its entry from `websterengine.CheckParent`'s own violation (`<class>: <detail>`) instead of a hand-copied string, so every committed-route row now depends on the text `fabricReferenceDetail` writes.
  No new test.
  Checked by mutation: changing `fabricReferenceDetail`'s `%q` to `%s` made `TestAcceptBatchFabricReference_ClearsCommittedReadOnly` fail; the change is reverted.
  The literal entry in `internal/webstercli/acceptaudit_test.go` stays, since one derived site pins the coupling.

## Verification after the last fix

`go build ./...`, `go vet ./...` and `go vet -tags integration` over the touched packages pass.
`go test ./...` passes with no failing package.
`lyx loom lint-comments` passes.

## Prior-round check

The review was saved before any prior-round file was read.
Round 2's fixes hold: the recover-batch refusal names both routes, the usage line names both, and the three tokencount rows exist.
Round 2's F1 fix was the origin of F1 here; its own suggestion (refuse a first character that is a quote or a backslash) was wider than the first-character test that landed.
Round 1 and round 2 deferred two items, neither in this round's write surface: the 11 MB `tools/tokencount/tokencount` binary still sits in the history of `340ed2e13` and `a39151dac`, and the fabric repo's live `_lyx/config/batcher.yaml` still carries `startup_context: 60000`, which `batcher.Active` now refuses.
Both stay for `ly:orch` or the operator.

## Deferred

Nothing from this round.
