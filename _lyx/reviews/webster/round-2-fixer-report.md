# Round 2 fixer report

## Fixed

- **F1** (BLOCKING, consistency): joined the mid-clause wraps in the changed comments of `internal/fabricengine/checkout.go`, `internal/fabricengine/mutation.go`, `internal/fabricengine/reconcile.go` and `internal/landingshed/finalize.go`.
  The `reconcile.go` file doc's changed sentence is now two lines, broken after `valid paired topology:`.
  Committed as `fabricengine, landingshed: fix F1 — join mid-clause comment wraps in the changed sentences`.
- **F2** (BLOCKING, consistency): the `ErrRemoteLeftover` doc keeps its list of fragments on the colon's line.
  Committed as `fabricengine: fix F2 — keep the ErrRemoteLeftover doc's list on its colon's line`.
- **F3** (BLOCKING, consistency): broke the changed sentences that held two independent clauses on one line, at the semicolon or after the comma before the conjunction, in `fabricengine/doc.go`, `add.go`, `weftbranch.go`, `configchanges.go`, `reconcile.go`, `remoteleftover.go`, `landingshed/doc.go`, `deps.go`, `finalize.go`, `finalize_integration_test.go` and `fabriccli/adoptremote_integration_test.go`.
  Wording is unchanged.
  Committed as `fabricengine, fabriccli, landingshed: fix F3 — break changed sentences at their independent-clause boundaries`.

No code or test changed.
`go build ./...` and `go test` pass over `internal/fabricengine`, `internal/landingshed` and `internal/fabriccli`, untagged and with `-tags integration`.

## Deferred

Nothing deferred.
A few long changed sentences in `fabricengine/doc.go` (for example the first half of the `Reconcile` sentence, `:` followed by a clause that runs on) still carry a clause break I judged a compound predicate, not an independent clause, and left on one line.
