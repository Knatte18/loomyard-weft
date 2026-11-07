```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens over the F1 test (`bootwindowlog_test.go`) and the F2 doc edit.

**Findings**

1. **LOW** — `bootwindowlog_test.go`, row `AFailedReadLogsNoInfoLine`: it passes even if `logBootWindowLocked` drops its `err != nil` check.
   - The row scripts `out == ""` together with `err`. `strings.SplitN("", " ", 3)` gives one field, so `len(fields) != 3` already rejects it.
   - The `err` branch is therefore not isolated. Deleting `err != nil` from `lifecycle.go:570` leaves every row green.
   - Fix: give the failing row a well-formed answer such as `"@0 %1 bash\n"` together with `err`, so only the error check can suppress the Info line.

Everything else holds:
- The fake matches `display-message` by its last argument, which is `bootWindowFormat`, so `answerFormat` reaches the call.
- Removing the Info call, or logging it when the field count is not 3, fails the success and single-token rows.
- `logBootWindowLocked` has no return value and no caller that depends on one, so "never fails the boot" holds by construction. The F1 fix text suggested also asserting `booted=true` through `ensureServerAndSessionLocked`. The test does not do that, and `lifecycle_test.go` says that path is out of hermetic reach, so I raise no finding.
- The `doc.go` edit changes comment text only, with no code effect. It matches the code: the detached-window line logs no pane id, so "where the open reports one" is accurate.
- The `name="my shell"` quoting assertion depends on the logger's text format. The tier-0 gate passed it (0 failed), so it matches.

**Tool calls:** none. Everything I needed was already in context.
