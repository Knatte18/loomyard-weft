```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set (`doc.go` window-order lines, new `bootwindowlog_test.go`) for agreement with its surroundings.

**Findings**

1. **LOW**, `internal/reedengine/doc.go` lines 663-667 (new text) against line 669 (unchanged).
   - The new text says the boot's first window "starts named after its shell and the shell-window step renames it shell".
   - Line 669 still reads "A fresh boot names its one window that way", meaning named `shell`.
   - A reader sees two accounts of the same window: named `shell` at boot, or named after the shell and renamed later.
   - Evidence: `lifecycle.go:624` runs `new-session` without `-n`. The review's F2 text says the shell-window step does the rename. Line 669 was not reworded to match.
   - Suggested wording for 669: "A fresh boot's one window ends up named that way."

2. **LOW**, `bootwindowlog_test.go`, the test name and doc comment ("NeverFailsTheBoot", "leaves the caller unaffected").
   - The test calls only `logBootWindowLocked()`, which returns nothing. It never drives `ensureServerAndSessionLocked`, so it cannot observe `booted=true` or a nil error.
   - F1's `fix` asked for the non-fatal rows to assert that. The file header of `lifecycle_test.go` says that function is out of hermetic reach, so the narrower scope is defensible.
   - The name and comment still claim more than the test checks. The failure rows assert only that no Info line appears. They do not pin the Debug line the code comment promises.
   - Evidence: `lifecycle.go:567-575`, where the failure path logs at Debug and returns, and the test's `want == nil` branch.

No other twin was missed:
- The `doc.go` "pane id where the open reports one" wording matches the detached-window line, which logs no pane id.
- `spawn.go:328` and `ownwindow.go:439` carry `name` and need no `doc.go` change.
- F3 has no hermetic test driving those two paths, so leaving it as log-only agrees with the review's fallback.

**Tool calls:** none. My context already held every file I needed.
