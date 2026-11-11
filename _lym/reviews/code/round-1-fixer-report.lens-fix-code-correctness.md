```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the round-1 change set (commits a115887b8, 30c5e8592, 4b97a9946, 9acf412c8).

**MEDIUM — F2 coverage is incomplete for two converted lines**
- Changed text: `internal/fabricengine/anchorstop.go:171`, where `upstreamAnchorStop` passes `lymVerbDir(codeWorktree)`, and `internal/fabricengine/pull.go:442`, where `settleUnmovedWeft` passes `lymVerbDir(f.warpPath)` for the records conflict push step.
- Reverting either line to the bare root breaks no test. The only tests that reach them are `pull_integration_test.go` and `pushclasses_integration_test.go`, and their fixtures give no anchored directory.
- Evidence: the fixer's own attempt, an anchored pull fixture, showed `lymVerbDir` resolving the root there. The warp at `.../002` is not a hub sibling of the weft at `.../003/container/...`, so no marker is read.
- The new unit rows cover `stopWayForward` (rewritten, both anchor-stopped variants, dirty records), `weftDirtyStop` and the `lymVerbDir` fallbacks. Those fail if the `lymVerbDir` call is reverted.
- The two lines above are the gap against F2's explicit list ("the pull.go records conflict push step and the weftDirtyStop and upstreamAnchorStop lines"). `weftDirtyStop` is covered. The other two need a deferral with that reason in the fixer-report.

**No findings** on the code paths themselves:
- `BranchRewritten` now returns `lymVerbDir(b.CodeWorktree)`. `CodeWorktree` is set for code targets (`originreconcile.go:717` sets `codeWorktree: worktree`) and is copied to the outcome at `:890`, so the single return is correct.
- `weftDirtyStop` and `upstreamAnchorStop` still hold for their callers (`pull.go:199`, `pull.go:415`, `pull.go:450`, `pushclassified.go:647`). Signatures are unchanged. `lymVerbDir` falls back to the root on an error and returns empty for an empty root, so no caller breaks.
- The `export_test.go` wrapper `WeftDirtyStopForTest` builds a `Fabric` with only `warpPath` and `weftPath`. `weftDirtyStop` reads only those two fields, so it is safe.
- The tier-0 gate passed with the new rows.

Calls made:
1. `grep CodeWorktree originreconcile.go originreconcilemoves.go`: to confirm `CodeWorktree` is set for code branches; my context only had the review's claim.
2. `sed -n 195,200p originreconcile.go` plus `grep codeWorktree originreconcile.go` plus `grep NamedWorktree` in `lymcwd`: to confirm `codeWorktree` is set per target and to see how `NamedWorktree` reads the marker; my context lacked `lymcwd`.
