```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it.

**Per behaviour, the test that pins it**

1. **A resize re-lays out the window evenly (after a join or open).** This is pinned by `TestOwnWindow_RealTmux/ResizeLaysTheSharedWindowOutEvenlyAgain` (`ownwindow_integration_test.go`, around line 384).
   - The test narrows one pane to 10 columns and grows the window by 40 columns. It then waits for the widths to differ by at most 1.
   - Without the hook, tmux's own resize gives the whole growth to one pane. Without `pinEvenLayoutOnResizeLocked`, the difference would be well over 1, so the row fails.
   - It covers the `createOwnWindowPaneLocked` path (`spawn.go:217`) through `addGrouped`, which opens and then joins the window.
   - It does not distinguish "hook pinned on open" from "hook pinned on join". The second `addGrouped` re-pins the hook anyway. That is acceptable.
2. **The hook is re-pinned after a strand leaves (`relayoutSharedWindowsLocked`, `ownwindow.go:306`).** No test pins it.
   - The resize row never removes a strand.
   - If the line at `ownwindow.go:306` were deleted, the new row would still pass. A window that once had three panes and drops to two would then keep a hook from an earlier layout. The layout name in the hook is unchanged by a leave, so a real defect here would be limited to a hook that was never set.
   - **Finding (MEDIUM, design): no test pins the re-pin on the leave path.** The location is `ownwindow.go:306` (evidence: `ownwindow_integration_test.go`, the new subtest only adds). The fix is to extend the resize subtest, or add a row to the existing leave subtest, so that after a removal the window's panes also return to even widths on a window resize. Add three panes, remove one, narrow one pane, grow the window, and wait for even widths.
3. **A zoomed window is skipped (`#{?window_zoomed_flag,0,1}`).** No test pins it.
   - No test zooms the window, resizes it, and asserts the window stays zoomed.
   - Removing the `if-shell` guard would pass the suite.
   - **Finding (MEDIUM, design): the zoom skip is untested.** The location is `ownwindow.go:316`. The fix is to add a subtest that zooms a pane of a shared window, grows the window, and asserts `#{window_zoomed_flag}` is still `1`. Do this on non-Windows only.
4. **Layout name quoting in `evenLayoutResizeHook`.** It is covered only indirectly. The integration row runs with the default `window_split`, so a different `window_split` direction is not exercised. This is low value because the layout names are fixed tmux tokens.
5. **Windows pins nothing (`runtime.GOOS` guard).** It is exempt: the new subtest returns early on Windows, and nothing asserts that no `set-hook` is issued. The guard is a one-line platform branch, so this is not worth a row. I raise no finding.
6. **A failed hook pin is logged and does not fail the step.** No test pins it.
   - The existing fake-tmux tests do not cover `pinEvenLayoutOnResizeLocked` (the grep found no hit in any `_test.go`).
   - **Finding (LOW, design): no fake-tmux unit row pins "a failing set-hook never fails the open or the removal".** The location is `ownwindow.go:328`. The fix is to add a row where the fake `set-hook` fails and the join or leave still succeeds. Only do this if a fake tmux seam already exists for these paths.
7. **A skipped layout (invalid `window_split`) also skips the hook.** `relayoutSharedWindowsLocked` returns early at `ownwindow.go:299`, and `createOwnWindowPaneLocked` returns an error before the pin. The existing invalid-config tests exercise the early return, but none asserts that no hook is set. This is covered by construction, so I raise no finding.
8. **The hook does not collide with the shell window's `window-resized` unpin (`windowsize.go:127`).**
   - Both set and unset target different windows. The unpin targets the shell window (`target`).
   - No test asserts that a group window's hook survives `pinGeometryOptionsLocked`. The existing `windowsize_test.go` asserts only the unpin calls on `target`.
   - **Finding (LOW, design): no test pins that the shell-window unpin leaves a group window's hook alone.** The location is `windowsize.go:127-128`, where the unscoped `set-hook -u -t target` is session-or-window-resolved.
   - Check this by hand: `set-hook -u -t <target>` without `-w` could resolve to the session's current window.
   - The fix is to extend the integration row to call the attach pre-flight, or `pinGeometryOptionsLocked`, after the pin and assert the resize still evens the panes.

**Tool calls**
1. `grep` over `internal/reedengine/*_test.go` for the hook names, `pinEvenLayoutOnResizeLocked` and `window_zoomed_flag`. My context did not show whether any existing test covered the new hook.
