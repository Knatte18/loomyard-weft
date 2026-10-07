# Round 1 fixer report

## Fixed

- **F1** (BLOCKING, consistency): rewrote the Go comments this diff added or changed in `internal/fabricengine/checkout.go`, `internal/fabricengine/mutation.go`, `internal/fabricengine/reconcile.go` and `internal/landingshed/finalize.go` to one sentence per line, and split the two-sentence line in `configChangeNotice`'s doc comment.
  Only wording-neutral line breaks changed; no code or test changed.
  `go build`, `go vet` and `go test` over `internal/fabricengine` and `internal/landingshed` pass.
  Committed as `fabricengine, landingshed: fix F1 — semantic line breaks in comments this diff changed`.

## Deferred

Nothing deferred.
