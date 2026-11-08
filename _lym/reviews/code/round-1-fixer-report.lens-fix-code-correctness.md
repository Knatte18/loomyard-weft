```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the three test edits in `internal/reedengine/ownwindow_integration_test.go` (commit 7499bb1f) by running each through concrete inputs.

**1. MEDIUM** — the F3 change does not fail with the defect in place.
- Changed text: the `LeavingASharedWindowRelaysItOut` subtest, with its 3-pane group, the removal of `third`, and the resize followed by `panesEven`.
- The finding asks for a test of the `e.pinEvenLayoutOnResizeLocked(window, layout)` call at `ownwindow.go:306`.
- Delete that line and the test still passes. The join path already pinned the same `layout` on this window, because `window_split` stays `width` throughout.
- The hook text is identical before and after the leave, so the resize assertion cannot tell the re-pin from the earlier pin.
- The review itself notes the line changes behaviour only after a `window_split` change.
- A test that does pin it would change `e.cfg.WindowSplit` between the join and the leave, use `restoreConfig(t)`, and assert the new split's geometry after the resize.
- Today the new assertions cover only the plain relayout from `select-layout`, which was already covered.

**2. LOW** — the F2 zoom test may pass with the guard removed.
- Changed text: `ResizeLeavesAZoomedSharedWindowZoomed`, which reads `#{window_zoomed_flag}` straight after `resize-window` returns.
- With `if-shell -F '#{?window_zoomed_flag,0,1}'` removed, the hook runs `select-layout`, which unzooms.
- Nothing in the test waits for the hook to fire. If tmux has not yet run the hook when `display-message` arrives, the flag still reads 1 and the test passes with the defect in place.
- The review says the hook is queued before the next client command, so the race is probably benign.
- A negative assertion with no wait cannot confirm it. Using a probe that proves the hook already fired would make it deterministic. For example, a second unzoomed group window, or `resize-window` followed by a command that reads state the hook changes.

**3. LOW** — `numberOf` and `panesEven` call the outer test's `t`.
- Changed text: `numberOf`, which calls `t.Fatalf`, and `panesEven`, which calls it. Both capture the parent `t` from `TestOwnWindow_RealTmux`.
- Subtests call them from the subtest goroutine, so a failure ends that goroutine through the parent's `FailNow`.
- The old inline `width` helper used the subtest's `t`.
- The neighbouring `display` helper already does the same, so the pattern is consistent.
- A failure there is still reported and the run still fails. This is a code smell, not a wrong result.

Checked and fine:
- **F1:** The `runtime.GOOS != "windows"` guard compiles, and `runtime` is still imported.
- **`window_share_max`:** The explicit value 3 passes `validateWindowConfig`, which only rejects negatives.
- **Later subtests:** The extra "zoomed" and "leaving" windows do not break the pane counts or the newest-window logic in later subtests.

Tool calls: none. My context already held the changed file, the review, the rubric and `ownwindow.go`.
