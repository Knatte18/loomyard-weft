# Round 1 fixer report

All four findings in `round-1-review.md` are fixed, one commit each, nothing pushed.

## Fixed

- **F1** (`a085f0971`): rewrote the new and changed comment lines in `internal/gitrepo/doc.go`, `internal/webstercli/cli_test.go` and `internal/websterengine/beginbatch_test.go` with one sentence per line, as `PATTERN-comment-line-breaks` requires; untouched legacy lines stay as they were.
- **F2** (`a4887d31e`): removed "idempotently asserts Master's model for this batch" from the bracket-verbs section of `internal/websterengine/doc.go`, so it agrees with the "one model per run" section.
- **F3** (`e61fb8997`): `contracts/stencils/loom/loom-rubric-plan-review.md` no longer states a check count; it says every check ID but `plan-unapproved` is enforced by the gate.
  It keeps the `format-unrecognized` through `commit-subject-mismatch` range because `contracts/stencils/claims_test.go` pins that name in the stencil, and that range is still the span of the checks `internal/planparser` emits.
  `go test ./contracts/...` passes.
- **F4** (`6b341f2b8`): `Publish`'s rejected-push reason, when the remote read succeeds and returns no commits, now says the remote holds nothing the local branch lacks, so a merge adds nothing and the remote rejected the push for its own rule, and the way forward is to clear that rule and resume with `lyx loom start`.
  The merge way forward stays for a non-zero count and for a failed read.
  `refusal-spec.md` splits the `Publish: push rejected` row into the commits case and a new `Publish: push rejected by a remote rule` row, `internal/landingshed/doc.go` states the new case, and `TestPublish_StuckBeforePullRequest` gains the zero-commit row; this is a row of the existing table, not a new test.

Checks after the fixes: `go build ./...`, `go vet ./internal/landingshed/...`, `gofmt -l`, `go test ./internal/landingshed/... ./internal/loomcli/... ./contracts/...` and the same landingshed and loomcli packages under `-tags integration` pass.
An intermediate commit of F3 failed `TestStencilClaims`; I amended it before moving on, so no commit in the branch fails that test.

## Deferred

Nothing deferred.
