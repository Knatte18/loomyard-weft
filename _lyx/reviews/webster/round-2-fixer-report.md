# Round 2 fixer report

All five findings in `round-2-review.md` are fixed, one commit each, nothing pushed.

## Fixed

- **F2** (`666e31a6e`): `internal/websterengine/state.go`'s file header no longer lists "last-asserted model"; the changed sentence is on one line.
- **F3** (`593aa6b30`): `contracts/stencils/loom/loom-rubric-plan-review.md` now names `delete-before-reference` among the checks this round's gate enforces, and names `delete-target-gone` beside `plan-unapproved` as a check the gate does not run, since it fires only at Webster's dispatch.
  The `format-unrecognized` through `commit-subject-mismatch` range stays as the span of the `internal/planparser` checks, because `TestStencilClaims` pins the name `commit-subject-mismatch` in this stencil.
- **F4** (`58525f095`): when `RemoteOnlyCommits` returns an empty tip, `pushRejectedReason` now says "the remote has no task branch" and keeps the remote-rule way forward.
  `refusal-spec.md`'s "Publish: push rejected by a remote rule" row and `internal/landingshed/doc.go` name the absent-branch case.
  `TestPublish_StuckBeforePullRequest` has a new empty-tip row; it is a row in the existing table, added for the coverage gap the finding names.
- **F5** (`bf7638727`): `backgroundShellSentence` is gone; `expiredShellWarning`, the name card 8 gives this wording, now holds the body, and the friction note calls it.
- **F1** (`b2820b0a2`): every new or changed comment sentence the finding lists that broke mid-phrase, or at a comma with no independent clause after it, is now on one line.
  Where a joined sentence shared a physical line with a legacy sentence, the legacy sentence now starts on its own line and keeps its old wrapping after that.
  New spec rows 23, 30 and 31 in `contracts/specs/loom-plan-spec.md` now hold one sentence per line, on indented continuation lines.
  On a closer reading during the fix, seven listed sites break only before an independent clause, which `PATTERN-comment-line-breaks` allows, so they are unchanged:
  `internal/webstercli/beginbatch.go:152`, `internal/loomcli/wiring_publish_integration_test.go:3`, `internal/planglyph/planglyph.go:86`, `internal/websterengine/rebaseline_test.go:432`, `internal/websterengine/recordbatch.go:120`, `internal/websterengine/runlevel.go:710` and `internal/webstercli/verbs_test.go:681`.
  The F5 rewrite already covered `internal/websterengine/triage.go:77`.

Checks after the fixes:
`go build ./...`, `go vet ./internal/... ./cmd/...`, `gofmt -l` over the changed Go files, `go test ./...`, and `go test -tags integration` over gitrepo, fabricengine, landingshed, loomcli, webstercli, websterengine, planglyph, planparser and cmd/lyx all pass.
No commit touches a `_lyx` path.

## Regression check against round 1

Every round-1 fix still holds at HEAD.
Round 1's F2 doc.go bracket-verbs wording is unchanged; its F3 rubric edit is extended by this round's F3; its F4 zero-commit branch is kept, and the empty-tip case is now its own branch.

## Deferred

Nothing deferred.
