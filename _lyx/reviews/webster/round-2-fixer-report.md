# Webster round 2 — fixer report

Every finding in `round-2-review.md` is fixed, each in its own commit.
After the last commit, `go build ./...`, `go vet -tags integration ./internal/... ./cmd/...`, `go test ./...` and `go test -tags integration ./...` all pass.
No round-1 fix regressed: the `verifytree` HEAD-moved guard and `-z` porcelain parse, the warp-only `MergeUntrackedFiles` and the reworded retired-mechanism prose are all still in place.

## Fixed

- **F1 (LOW)**, `a8249b6f8`: each semicolon that joined two independent clauses on one new comment line now ends its line, in `cardgate.go`, `verifygatereport.go`, `websterengine/doc.go`, `runlevel.go`, `runlevel_test.go`, `fabricengine/mergestage.go` and `fabricengine/mutation.go`.
  `shell/pwsh.go`'s appended clause became its own sentence, `Source builds on it.`.
  `KindMergeTrackedStaged`'s doc in `mutation.go` is now one sentence or clause per line.
- **F2 (LOW)**, `bb1dcf361`: every wholly new comment block F2 lists is rejoined to one sentence per line, breaking only at sentence and independent-clause boundaries.
  Only lines the diff wrote were touched.
  Legacy paragraphs the review excluded are unchanged.
- **F3 (NIT)**, `d97942bcb`: the new markdown lines in `CLAUDE.md`, `CONSTRAINTS.md`, `loom-plan-spec.md`, `webster-spec.md`, `webster-body-verify-fix.md`, `webster-template-master.md` and `docs/overview.md` now break after the semicolon.
  List-item continuations keep their indentation.

## Deferred

None.
