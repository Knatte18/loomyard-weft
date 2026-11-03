```yaml
lens: fix-code-correctness
outcome: reported
```

No findings on the engine tests; one MEDIUM on the F2 fix.

**fix-code-correctness: findings**

- **MEDIUM** — `internal/webstercli/verbs_test.go`, `newVerbsFixture` (the new `plan := onlyCreatePlan(...)` with `plan.Root = layout.AnchorRel`). The begin-batch dispatch check is still not pinned.
  - The review's fix asked for "card targets that exist under" the anchor. The fixer kept a Create target, `internal/only/new.go`.
  - `path-missing` is a ref check on Edit and Delete targets. A Create target that does not exist is not flagged.
  - If `index.ValidateDispatch` at `beginbatch.go:480` resolved against the anchor again, `anchor/backend/internal/only/new.go` would also be absent. That is the same outcome as at the worktree root, so `TestBeginBatchCmd_HappyPath` stays green.
  - I did not run that mutation. I confirmed only that `path-missing` is a ref check (`internal/planglyph/doc.go:37,152`).
  - The `RecordBatch` `DoneChecks` side is pinned. The first gate run failed with `Create target "backend/internal/only/new.go#" still does not…` once the committed path was not root-prefixed.
  - `ValidateDispatch` in begin-batch, `runlevel.go:648` and `DiskSizes` (`cli.go:292`, `runlevel.go:511`, `rebaseline.go:110`) remain unpinned.
  - Fix: add an Edit target that exists under `backend/` and is committed in the fixture repo, for example a second card or an Edit group. With the anchor-based resolve restored, begin-batch then fails with `path-missing`.

**Checked, no finding**
- `writtenWorktreePaths` split row: `a.go` becomes `backend/a.go`, `../top.go` becomes `top.go`, and `../../outside.go` is dropped, giving `[backend/a.go top.go]`. Swapped roots would return `a.go`, so the row fails with the defect.
- `suspect_test` subtest: `trackedRel` returns `backend/f.txt` and `tracked.txt`. The `WorktreeBlob` path is `Join(worktree, rel)`. Swapped roots fail the within-tree check for `../tracked.txt`, and the blob would come back empty. The git probe on the pane cwd is not covered, which is acceptable.
- `batchfail` row: `x.go` resolves under `internal/`, so the blob equals the forged-content hash. Swapped roots give an empty blob, so the row fails with the defect.
- The `verbs_test.go` edits are consistent. The `Envelope` and `RunningThenTerminal` commit paths and the two rebaseline plans (`Root: anchorRel`) all match the new root. No other caller broke; the second gate run was green.

**Further tool calls**
1. `grep -rn "path-missing" internal` and `sed -n 470,490p internal/websterengine/beginbatch.go`: confirm what `ValidateDispatch` checks for Create targets, which my context lacked.
