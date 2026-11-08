---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/reedengine/ownwindow_integration_test.go:385
    summary: The resize subtest is registered on Windows despite being platform-bound
    fix: Wrap the entire ResizeLaysTheSharedWindowOutEvenlyAgain t.Run call in a runtime.GOOS != windows condition, using the Go string literal for windows, and remove the early Windows return from its callback. Keep the existing assertions inside that callback.
---

### [BLOCKING] Exclude the platform-bound subtest before registration

**Location:** internal/reedengine/ownwindow_integration_test.go:385

**Issue:** The new subtest calls `t.Run` on Windows and then immediately returns without exercising any behavior.
`pattern/PATTERN-test-speed.md`, under “No test skips by design,” requires an OS-bound subtest to be excluded before `t.Run`.
Returning instead of calling `t.Skip` still registers the prohibited platform-bound case and reports an empty success.
The constraint directive makes every violation of a listed constraint BLOCKING, regardless of its size.
This rule is explicitly enforced by review discipline rather than a mechanical gate.

## Rejected

- Hook collision: `pinEvenLayoutOnResizeLocked` sets a window-scoped hook using a pane or window ID.
  The production readers of `windowResizedHookName` are that setter and `pinGeometryOptionsLocked`.
  The latter unsets the shell window's hook and the session hook, leaving another window's local option intact.
  Its callers are fresh boot and attach preflight; attach resolves the shell window through `strandWindowTargetFor`.
  The other symbol readers are assertions in the window-size, lifecycle and attach tests.
- Missing layout paths: `createOwnWindowPaneLocked`, called by `launchStrandLocked`, pins after shared-window creation or join.
  Ungrouped and separately placed strands return before the layout and pin.
  `relayoutSharedWindowsLocked` pins after each removal layout; its callers are `RemoveStrand` and `ReplaceStrand`.
  Invalid window configuration returns before those layouts and pins.
  The new pin helper has only these two callers, and `evenLayoutResizeHook` has only the pin helper as its caller.
- Hook construction: layout names come from the closed mapping in `windowSplitLayout`.
  The nested tmux command is quoted as one `if-shell` argument, and `-F` evaluates the zoom format without spawning a shell.
  The condition suppresses layout selection when zoomed.
  The installed tmux manual confirms that setting a hook without an array index replaces its entries.
  Runtime confirmation of event targeting and zoom preservation remains unavailable for the reason below.
- Scope and prose: the diff implements the Sketch's reed-layout approach and does not change Codex's `--no-alt-screen`.
  The Windows guard, replacement semantics and zoom condition support the corresponding doc and commit claims.
  `_lym/landing/summary.md` describes the same implementation.
- Duplicate test: `GroupOfTwoSharesOneWindowSideBySide` checks initial placement, not equality after resizing.
  `LeavingASharedWindowRelaysItOut` checks the remaining pane after removal.
  `TestMultiplexerContract` checks hook-array behavior and a height-pinning hook, not reed's new even-layout hook.
  None duplicates the new regression's assertion.
  Its bounded polling checks immediately and waits only while the result is pending; it does not wait out `sleep 300`.

## Focus departures

- “ownwindow_integration_test.go `ResizeLaysTheSharedWindowOutEvenlyAgain`: does it fail without the hook”:
  I could not execute the regression or confirm failure without the hook.
  `timeout 3600 lym gate test --tier 2 ./internal/reedengine` failed before tests started.
  The gate could not write its holder record under the hub's `_bolt/.lym/gate` directory because it is read-only here.
  Run-process registration also failed on the same filesystem restriction.
  I did not bypass the required gate or modify target files to perform a negative-control run.
  No passing test run or live hook validation is claimed by this report.

Reviewed the committed range `539525a02bc86d7d4ef3a6afdf95989cdc52080d..HEAD` and its commit messages.
