```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. Each case below is run with `WorktreeRoot=/wt` and `PaneCwd=AnchorRoot=/wt/backend`, unless noted.

**Per function or branch (case run, verdict)**
- `hubgeom.WebsterGeometry`: anchor `backend` gives `WorktreeRoot=/wt`, `PaneCwd=/wt/backend`; anchor `.` makes them equal. Correct.
- `standalonegeom.WebsterGeometry`: both are `target`, the same values the old `WorktreeRoot` and `RepoRoot` held. Correct.
- `BeginBatch` (`beginbatch.go:571`, `:480`): `HeadSHA` and `ValidateDispatch` use `WorktreeRoot`, so go-git opens `/wt`, which fixes #32. Correct.
- `BeginBatch` (`beginbatch.go:521`, `:589`): the plan display uses `Rel(PaneCwd, PlanDir)`, which gives `_lym/plan`, and `{{.worktree_root}}` is `PaneCwd`, as before. Correct.
- `RenderRecoveryPrompt` and `RenderMasterPrompt` arguments (`recoverbatch.go:318`, `runlevel.go:756`): the argument order matches the signatures and the old order, with `repoRoot=WorktreeRoot` and the prompt root `PaneCwd`. Correct.
- `trackedRel` with relative `p="src/a.go"`: it resolves to `/wt/backend/src/a.go`. The within-tree check and `IgnoredPath` run against `/wt`, and the relative path is `backend/src/a.go`, which `git rev-parse <commit>:<path>` expects. Under `/wt/backend/_lym` it returns `ok=false`. Correct.
- `suspectBlobs`: `WorktreeBlob(worktree, Join(worktree, rel))` is consistent with the root-relative `rel`. With empty `paths` the loop does nothing, so the `PaneCwd=""` sites are harmless. Correct.
- The `failBatchInput` site at `recordbatch.go:201` sets no `PaneCwd`. It also sets no `SuspectPaths`, so `suspectBlobs` never resolves anything through the empty `PaneCwd`. Correct, though fragile.
- `checkSuspectPaths`: a path under `/wt/other` lands in the default branch and is probed with `IgnoredPath` and `PathDiffers` at the git root. The anchor-`_lym` cases stay unverifiable. Correct.
- `uncheckableReason`: the `.lym` and `_lym` checks use `PaneCwd`, and the outside-worktree check uses `/wt`. Correct.
- `ClassifyViolation` (`audit.go:404-443`): `OtherWorktrees` and `IgnoredPath` now run at `/wt`.
  - A write at `/wt/sibling/x` was policy under the old anchor root and is now correctness unless git-ignored. This follows the record, which makes the worktree root the task worktree, and I count it as intended.
- `runOwnPath` and `UncommittedPaths` (`auditledger.go:273-292`): `DirtyPaths(/wt)` returns repo-root-relative paths, and joining them onto `WorktreeRoot` is now consistent. It was inconsistent in a subpath hub before. Correct.
- `batchFabricReferenceEvidence`, `reset.go` git probes, `planfixer.go`, `digest.go`, `archive.go` and `verifygate.go` git closures: all read `WorktreeRoot`, which is the git root. Correct.
- `loadRunWrites`, `writtenWorktreePaths`, `contractCanon`, `CheckParent`, `CheckFork` and `AuditForks*`: all resolve against `PaneCwd`, the cwd the transcript was recorded under. `writtenWorktreePaths` canonicalises `root` from `worktree` and resolves writes from `paneCwd`, so the within-tree drop and the relative path are measured against `/wt`. Correct.
- `MerriamBase`: instruction files are read from `PaneCwd`, as before. The PATTERN directive reads `WorktreeRoot`, which equals the old `RepoRoot`. Correct.
- `RecordBatch` (`recordbatch.go:136`, `:933-1067`): `DoneChecks`, `Delta`, `DetectDrift`, `LaterDeleteReferences` and `DiskSizes` use `WorktreeRoot`. These are the git and plan-path calls, which fixes #31. Correct.
- `RecordBatch` (`recordbatch.go:532`) and `verify.go:68`: the card-verify runner and the verify gate run in `PaneCwd`, as before. Correct.
- `standaloneKeepReset` (`webstercli/reset.go:224`): this is standalone-only and `WorktreeRoot` is `target`, so it is unchanged. Correct.
- `RepoRoot` removal: no non-test Webster reader is left. The remaining `RepoRoot` hits are burler's and gatefixer's own structs. Correct.

**Findings**
None. I found no wrong result, swallowed error or inconsistent failure-path state in the changed code.

One thing I could not settle from my context, so I record it as an observation and not a finding: `verifygate.go:255` reads `go.mod` through `readModulePath(WorktreeRoot)`, which is now the git root and was the anchor before. The record says `go.mod` sits at the worktree root. A subpath-anchored hub with its `go.mod` at the anchor would resolve differently, but I found no evidence of one.

**Tool calls**
1. `grep` for `failBatchInput{`, `readModulePath` and `SuspectPaths:` in non-test `websterengine`, because my context lacked every construction site of `failBatchInput` and the `go.mod` reader's definition.
