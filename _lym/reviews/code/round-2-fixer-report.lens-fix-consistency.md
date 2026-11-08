```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 test-row changes for consistency with their siblings.

No findings.

- **F1 row:** `ConfirmEntryHandedTheTestBinaryRaisesNothing` copies the `exeErr` row's `wantMessage`, `anchorMoved` and the no-prompt expectation. It adds `wantLog`, so the revert-to-bare-`executablePath()` case fails the log and prompt checks. The `testBinary` and `wantLog` fields are documented. The formatter only realigned the struct fields.
- **F2 loomcli row:** it uses a zero `loomCLI`, so `lymBinary` is nil. That matches the field's documented "nil means selfexec.LymBinary".
- **F2 window row:** the `defaultLookup` row keeps `wantCalls` at 0 and expects `window: resolve the running lym`, which matches `internal/reedcli/window.go:90`. The stand-in lookup is still set for every other row, and its check on `lymPath` still applies to them.
- **Twins:** the fronttask `startAction` default has no test row, which the review excluded.

Further tool calls: one `git diff HEAD~2 HEAD` on `internal/reedengine/buttonclick_test.go`, to see the formatter's realignment, which my context lacked.
