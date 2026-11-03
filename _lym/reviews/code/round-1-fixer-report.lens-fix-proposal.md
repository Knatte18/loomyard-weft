```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals behind F1 and F2 as the fixer carried them out.

**F2 (MEDIUM): the begin-batch half of the proposal does not pin what the review says it pins**

- Changed text: `newVerbsFixture`'s plan, now `onlyCreatePlan("", "internal/only/new.go")` with `Root = "backend"`. It sits in `internal/webstercli/verbs_test.go`.
- The plan has only a Create target, and `checkPathMissing` skips Create refs (`internal/planparser/validate.go:1300-1301`). It reports nothing for this plan wherever the root resolves, so the review's predicted "path-missing at the dispatch boundary (beginbatch.go:480)" cannot happen.
- Moving `ValidateDispatch` back to the anchor makes it look for `backend/backend/internal/only/new.go`, and moving it back to the worktree root makes it look for `backend/internal/only/new.go`. Neither exists, so a Create-existence check passes both ways. I did not read `ValidateDispatch` itself.
- `TestBeginBatchCmd_HappyPath` therefore stays green with `beginbatch.go:480` and `runlevel.go:648` moved back to the anchor.
- The same holds for `DiskSizes`, `Delta` and `DetectDrift` on a nonexistent Create target. Only `DoneChecks` in `TestRecordBatchCmd_Envelope` is demonstrably pinned. There the Create target must exist after the commit, and `backend/backend/...` would not.
- Evidence the pin is partial: the fixer's gate run stayed green with the begin-batch tests unchanged apart from the fixture.
- A sound case would be an Edit target that exists only under `backend/`, for example `backend/README.md` as `TestPersistentPreRunE_PlanDirAnchoredAtSubpath` does. A Create target satisfied nowhere leaves the dispatch paths unpinned.

**F1: no findings**

- `writtenWorktreePaths`: the subdirectory `paneCwd` row would fail if the two roots were swapped. With `root` as `backend`, the write `a.go` resolves outside it and is dropped. `../top.go` is covered, and the `../../outside.go` write is dropped as outside the tree.
- `trackedRel` and `suspectBlobs`: the swapped call returns `ok=false` for `f.txt`, so the new subtest fails. The subtest does not depend on earlier subtests: `tracked.txt` is committed, so the worktree and HEAD blobs match, and `-run` in isolation works too. The `_lym` branch under `paneCwd` stays untested, but the review did not ask for it.
- `failBatch`: with `PaneCwd = repo/internal` and path `x.go`, swapped roots resolve `x.go` against `repo/internal` as the worktree, so the path falls outside the tree and the blob comes back empty. The assertion fails.
- The F1 proposal added subtests and a row instead of new test functions, as the review required.

Further tool calls:
1. `grep path-missing`, `ValidateDispatch` in `internal`: to find where each is emitted and defined; my context held neither.
2. `sed` of `internal/websterengine/beginbatch.go`, a path I guessed wrong, so it returned nothing.
3. `Read` of `internal/planparser/validate.go:1290-1339`: to check whether Create refs are exempt from `path-missing`.
