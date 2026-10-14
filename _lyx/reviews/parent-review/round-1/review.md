---
verdict: APPROVED
---

## Scope against the board entry

The design answers the entry's three "Scope to settle" items and adds nothing outside them.

- Item 1 (done-check against a later rename) and item 2 (batch gate survives a rename): D1 splits a batch before a card that renames or deletes an earlier same-batch card's target.
  This is the route the entry lists under "Ways out to weigh" as "the Batchifier puts a rename in its own later batch", and the record names it as such in Scope.
  With the split, each card's handles are checked against a tree where they still stand, which meets item 1's intent, and the batch gate never names a directory its own batch removes, which meets item 2.
- Item 3 (pinned and repo-scanning tests caught at the card): D2 runs every `//lyx:guard`-marked root-module test in each card gate, after weighing `./cmd/lyx` per batch and a plan-time check, as the item asks.
  Both tests the entry names, `TestGitrepoBoundary_PinnedRunCallSites` (`cmd/lyx/gitrepoboundary_test.go:111`) and `TestTierPurity_UntaggedTestsSpawnNothing` (`cmd/lyx/tierpurity_test.go:91`), carry the marker today.
- The "verb to accept a batch by hand" the entry mentions is only an observation; Scope Out and Q6 leave it out, which the entry does not contradict.
- No direction the entry rules out is taken, and no work belongs to another board entry.
  The Go-only guard step leaves .NET coverage to `dotnet-scoped-verify`, recorded as an open risk rather than built here.

## Overlap with other runs

`lyx loom peers` lists no parent commits.
`own_row` is 3.

- `tender` (row 21): its changed paths include `cmd/lyx/seamsignature_test.go` and `cmd/lyx/testdata/test-budget.yaml`.
  This run's design names `seamsignature_test.go` only as an example of a compile-time pin and does not plan to edit it; `test-budget.yaml` is a shared bookkeeping file.
  `tender` is the run whose batches 01 and 07 exposed these gaps; it changes no webster, batcher or impactset code, so no behaviour conflicts.
  Landing order: `tender` (row 21) lands first, and this run takes its changes at Merge-In.
- `twb-fx-ia`, `twb-fx-ib`, `twb-fx-ic`, `twb-fx-id` (row 3): bench pairs replaying `intake-bugs-1012`, whose external-test mapping in `planparser.CardTargetDirs`, `gateScopeOf` and the batcher's read-set estimate already landed on `main` (`22b4aff8a`).
  This run touches `gateScopeOf`'s file and `internal/batcher` at other behaviour (batch boundaries, the guard step), so the overlap is text-only.
  The pairs are torn down after the bench and do not land.
  Landing order: a tie (both row 3).
- `twb-fx-wa`, `twb-fx-wb`, `twb-fx-wc`, `twb-fx-wd` (row 3): bench pairs replaying `run-window-bugs`; no overlap.
- `claimed-scope-edit` (row 7): darn's decision record and PR-Align; no overlap with webster, batcher or impactset.
- `tokencount-fork-output` (row 0): `tools/tokencount` only; no overlap.
