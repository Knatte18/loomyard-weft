```yaml
lens: fix-proposal
outcome: reported
```

Findings, by finding id. I ran a tool call only for the F7 question and to confirm `boundOneLine` is still used.

**MEDIUM — F7, `TestEnsure_UnparseableDriftYamlFailsTheWalkNamingTheFile` (`reinterpret_integration_test.go`)**
- Summary: the new row probably never reaches the `Rewrite` error branch the review named.
- Evidence:
  - It asserts only that the error contains `drift.yaml`.
  - `ReconcileHubWideAt` (`configsync.go:907`) and the parse sites at `configsync.go:249`, `:335` and `:792` read hub-wide files before `applyReinterpretations` runs in `reconcileHubWide`.
  - An unparseable `capture: {dir: [` therefore most likely fails in that earlier parse.
  - The `fileError` wrap at `reinterpret.go:55` (`reinterpretation %s: %w`) is not shown to be reached, since the assertion never checks for that text.
- Impact: the row passes whichever parse fails first, and that failure is the same one the existing failed-walk rows already cover.
- Fix: assert `"reinterpretation drift-dir-root-base"` in the error, or report in the fixer-report that the branch is unreachable through a file. The commit message and report should not claim the `Rewrite` branch is covered.

**LOW — F7, `noNotifier` row**
- Summary: the "no installed notifier" row cannot detect a notifier left installed by an earlier test.
- Evidence:
  - It uses `sent := new([]notice)` and installs nothing.
  - It relies on every earlier test having restored the global notifier.
  - If one leaked, the walk would call it, the row would still pass, and `notifyReinterpretations`' nil branch would go unexercised without anyone noticing.
- Fix: no production seam is needed. Mention the limit in the report.

**LOW — F3, `readLogTail`**
- Summary: when the read window starts exactly on a line start, the first complete line is dropped, which is harmless except when the log holds three or fewer lines.
- Evidence:
  - With `offset > 0` it always drops through the first `\n`. A window that begins mid-rune also decodes to U+FFFD if the window holds no newline. This only matters for very long single lines.
  - The "short last line after two long lines" row does not touch the 64 KiB window, so the seek, cut and drop path has no test.
- Fix: add a row with a log over `outputTailReadBytes`, or accept this.

**No finding — F1, F4, F5, F6, F8**
- F1: the two-outcome assertion plus the child-log pin holds. The glob needs exactly one `capture-child-*.log` and runs in the first subtest, so no earlier child log can exist. The `wantError` prefix matches `FailedCaptureError`'s format.
- F4: `reconcileNotifierFor` is a behaviour-preserving extraction. `TestRootPreRun_InstallsTheHubNotifiers` cuts at the first `PersistentPreRunE: func(`, which the earlier gate run showed lands on the root command's pre-run.
- F5: a quoted `"null"` string has tag `!!str`, so it is still treated as a literal directory name, which is right.
- F6: the timeout row's expected `{dir}` text matches what the code produces.
- F8: the new wording stands alone.
- `boundOneLine` is still used (`spawn.go:186`), so it is not dead code after the split.

Tool calls:
1. `grep boundOneLine(` over `internal`: confirmed the old helper still has a caller, which my context lacked.
2. `grep Unmarshal|…|ReconcileHubWideAt` in `internal/configsync`: checked whether hub-wide reconcile parses `drift.yaml` before `Rewrite`, which my context lacked.
