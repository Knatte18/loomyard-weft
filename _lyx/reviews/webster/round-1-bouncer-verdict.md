---
verdict: APPROVED
rationale: "All six round-1 findings are valid and fixed in one commit each (1205cacf4, 0161653f4, 9190ab11d, 884d19b50, 8f91a9bfd, 0aaccbfc9); go build, go test, go test -tags integration and go vet -tags smoke all pass, gofmt -l over the diff's Go files is empty, and the fixes add no new defect."
---

# Webster-Review round 1 — Bouncer verdict

The review range is correct.
`_lyx/shed/parent-review/status.json` gives `product.parent` = `main`, and no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` over merge base `218ba2a68`.
The report has no `## Focus departures` section, so there is no departure to ratify or reject.

## The findings

Each finding is a real defect in the diff, located correctly, and none falls under the rubric's `Do not flag` list.

- **F1 (BLOCKING): fixed in `1205cacf4`.**
  `discussionCommitPathspec(location)` now adds `LoomParentReviewDirRel()` only when `holdsFile` finds a non-directory entry beneath the parent-review directory.
  That is the same guard `statusCommitPathspec` applies.
  The test now covers three cases: the directory absent, an empty round, and a round with a request file.
  The new doc comment's claim that round files are never deleted is what makes skipping the tracked-but-removed case safe.
- **F2 (BLOCKING): fixed in `0161653f4`.**
  `resolvePersistentPreRun` now returns early through `inReviewGroup`, which walks up to a `review` command whose parent is `loom`.
  `TestResolvePersistentPreRun_SkipsReviewGroup` calls the loom parent's pre-run directly for the group and each verb over a non-git cwd, so it exercises the path `cobra.EnableTraverseRunHooks` takes in production.
  review.go's file comment now states the mechanism correctly.
- **F3 (MEDIUM): fixed in `9190ab11d`.**
  `wire`'s doc comment sits on `wire` again, and `gofmt -l` over every Go file in the diff reports nothing.
- **F4 (LOW): fixed in `884d19b50`.**
  `RecordVerdict` re-reads `request.json` under the round lock and writes `review.md` inside that critical section.
  `MarkExpired` refuses with `ErrVerdictRecorded` under the same `lockPath(r.Number)`.
  `readHeldJSON` avoids re-acquiring the held lock.
  The retries in `gate()` and `final()` terminate: the refusal means `verdict.json` exists, so the next `Latest()` read takes the verdict branch.
- **F5 (LOW): fixed in `8f91a9bfd`.**
  Every listed comment now breaks at sentence or clause boundaries, per CONSTRAINTS.md's Comment Line-Break Convention.
  A sweep of every added two-line comment run in the diff found no remaining mid-phrase fixed-column wrap.
- **F6 (NIT): fixed in `0aaccbfc9`.**

## Independent checks

I ran the plan's whole `verify:` block on HEAD `0aaccbfc9`: `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` all pass.
The working tree is clean.
I also read the shuttle wait-loop change (`handleGatedBoundary`, `evaluateGate(final)`) and the gate and final closures.
The turn-boundary flag, the memo clearing after a pending result, and the run-time refusals of `Final` and pending on a non-`PassOnCap` entry match card 1 and the Shared Decisions.

The per-card mechanical check holds as the report states.
The cards carry only Create and Edit groups, so no Rename AST check and no `assert-no-callers` applies.
