# Round 3 fixer report

All five findings in `round-3-review.md` are fixed, one commit per finding (two for F1), nothing pushed.

## Fixed

- **F1** (`88c2320d6`, `346144012`): every comment sentence the finding lists that held a changed line is now on one line, or broken only at a semicolon, a `, so` or a `, and` before an independent clause.
  The sites are `cmd/lyx/destructiveguard_test.go`, `internal/fabricengine/doc.go` (the `Kind` vocabulary and the `ResetPairWarp` sentence), `internal/planparser/doc.go`, `internal/websterengine/beginbatch.go` (file header, `BeginDeps`, `BeginBatch`, the pre-existing-report guard), `internal/websterengine/beginbatch_test.go` (two), `internal/websterengine/doc.go` (the backgrounded-fork paragraph and the crash-window sentence), `internal/websterengine/runlevel.go`, `internal/websterengine/runlevel_test.go`, `internal/webstercli/cli_test.go`, `internal/webstercli/wiring.go` and `tools/sandbox/suite.go`.
  The detector re-run after the first commit found one site the review had wrongly passed: `internal/gitrepo/doc.go`'s changed "See PATTERN-gitrepo-client-boundary …" sentence, whose first line the diff had split off a legacy wrap.
  That sentence and the `gitexec` sentence after it are reflowed in the second F1 commit.
  After the fixes, every multi-line sentence in a changed comment block that still holds a changed line continues at a `;`, `, so` or `, and` boundary, which `PATTERN-comment-line-breaks` allows.
- **F2** (`c1c600db2`): `internal/fabricengine/destroy.go`'s recording-contract comment says "every executor below", so the count no longer goes stale when an executor is added.
  The first two sentences of that paragraph are reflowed with it, since the changed line sat inside them.
- **F3** (`d96b6980c`): `internal/gitrepo/push.go`'s header says "All of them are push-shaped remote calls".
- **F4** (`d4a3484ae`): `contracts/stencils/loom/loom-rubric-plan-review.md` no longer bounds the gate-enforced checks at `commit-subject-mismatch`.
  It says every listed check is enforced by the gate except `plan-unapproved` and `delete-target-gone`, and keeps the line explaining that `delete-target-gone` fires only at Webster's dispatch.
  `contracts/stencils/claims_test.go` pinned the name `commit-subject-mismatch` in this stencil, which is why round 2 kept the range.
  Its claim now pins `` `delete-target-gone` fires only at Webster's dispatch `` instead, the sentence this rubric must keep.
  That corrects the existing claim row and adds no test.
- **F5** (`ef4941c68`): `contracts/specs/refusal-spec.md`'s "Publish: push rejected" row now names a remote task branch tip that could not be read as a second condition, which `pushRejectedReason` already handled and `TestPublish_StuckBeforePullRequest` already reaches.

Checks after the fixes: `go build ./...`, `go vet ./...` and `go test ./...` all pass; `gofmt -l` lists only files this round did not touch.
The integration tier was not re-run, since no fix changes code.
No commit touches a `_lyx` path.

## Regression check against rounds 1 and 2

Round 2's F2 (the `state.go` header), F4 (the empty-tip branch, spec row and test row) and F5 (`backgroundShellSentence` folded into `expiredShellWarning`) all hold at HEAD.
Round 2's F3 kept the `commit-subject-mismatch` range because a claims test pinned it; this round's F4 replaces that range and moves the pin.
Round 2's F1 listed seven sites that break before an independent clause and were left alone; they are unchanged and still compliant.
Round 3 focus item 5 names `backgroundShellSentence`, which round 2's F5 removed, so the directive's premise was stale and no change followed from it.

## Deferred

Nothing deferred.
