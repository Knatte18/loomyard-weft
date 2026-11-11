```yaml
lens: code-tests
outcome: reported
```

Task: list per behaviour the test that pins it, for the code-tests lens.

**Per behaviour: pinning test, or none**

1. `pushStep` for a code side and a records side, with a recorded anchor: pinned by `failedpush_test.go` `TestFailedPushRecord`. The rows are `AnchoredCodeTransportPushesInTheAnchoredDir` and `AnchoredRecordsAuthPushesInTheCodeWorktreesAnchoredDir`.
2. `codeDivergedWayForward` (merge at the root, push in the anchored directory): pinned by `AnchoredCodeNotFastForwardMergesAtTheRootAndPushesInTheAnchoredDir`.
3. `rewoundWayForward` (git steps at the root, push in the anchored directory): pinned by `AnchoredCodeRewoundResetsAtTheRootAndPushesInTheAnchoredDir`.
4. `lymVerbDir` with an empty `worktreeRoot` returning `""`: no row. The test has no anchored row with an empty `CodeWorktree`. A record with no code worktree is the case the doc comment promises.
5. `lymVerbDir` with an unreadable or unusable marker (Warn log, falls back to the root): no row. `readRecordedAnchor` returns an error for an invalid marker or a read failure, and nothing in the diff drives that branch.
6. `lymVerbDir` with no marker (falls back to the root): the existing unanchored rows pin it, since they use `/hub/...` paths with no marker on disk. `NamedWorktree` returns `"."`, so the output is unchanged.
7. `originreconcile.go` `stopWayForward` `BranchRewritten`, code side (now `lymVerbDir`): none. The grep of `*_test.go` found no `BranchRewritten` and no `fabric pull` assertion tied to an anchored worktree.
8. `anchorStopWayForward`, pair and prime variants (`lymVerbDir(b.CodeWorktree)`): none. The grep found no test asserting `lym fabric anchor ... in` with an anchored worktree.
9. `dirtyWayForward`, records side (`lym fabric commit` in `lymVerbDir(b.CodeWorktree)`): none. No anchored assertion appears in the grep output.
10. `pull.go` `settleUnmovedWeft` `ErrReplayConflict` push step (`lymVerbDir(f.warpPath)`): none. No `lym fabric push` in assertion with an anchored warp appears in the grep output.
11. `bolt.go:428`: unchanged, and it uses `pushStep` for the bolt side, which does not call `lymVerbDir`.

The four new anchored rows each exercise a distinct `pushStep` or way-forward path, so none is redundant with an existing row.

**Findings**

- **F-T1: MEDIUM, design.** `internal/fabricengine/originreconcile.go:383-385` (`stopWayForward` `BranchRewritten`, code side), `originreconcile.go:427` and `:429` (`anchorStopWayForward`) and `originreconcile.go:440` (`dirtyWayForward`, records side) have no test for their new `lymVerbDir` call. Add anchored rows to the existing way-forward tests for these paths, using the `anchoredHub` fixture pattern in `failedpush_test.go`.
- **F-T2: MEDIUM, design.** `internal/fabricengine/pull.go:442` (`settleUnmovedWeft`, records conflict stop) has no test that an anchored warp worktree puts its anchored directory in the push step.
- **F-T3: LOW, design.** `internal/fabricengine/failedpush.go:242-250` (`lymVerbDir`) has no row for an empty `worktreeRoot` or for an unreadable or invalid marker. Add a records-side row with an empty `CodeWorktree`, and an invalid-marker row asserting the root fallback.

Evidence: `failedpush_test.go` diff, lines +199 to +365, and `originreconcile.go:372-443`. The grep of `internal/fabricengine/*_test.go` for `BranchRewritten`, `fabric pull` in, and `lym fabric commit` in returned no hit that covers an anchored worktree.

Tool calls:
1. `grep` of `internal/fabricengine/*_test.go` for the changed way-forward strings, `BranchRewritten` and `AnchorFileName`. My context lacked which tests already assert the originreconcile and pull lines.
