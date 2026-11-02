```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Traced inputs:
- **F1 branch, `anchorOf` returns a missing `dir/backend`:** `os.Stat` errors, the click shows "the anchor directory … is gone" and returns. No prompt is raised and nothing runs, for plain and confirm entries alike.
- **F1 branch, `runDir` is a file:** the `IsDir` check rejects it with the same message. The wording is slightly off but harmless.
- **F1 branch, `anchorOf` fails:** the earlier error branch still returns before the new stat. A confirmed click with a mismatched hash and a gone anchor reports the anchor first, which is acceptable.
- **F3 Info log:** only the message text changed. The condition `runDir != spec.Dir` is unchanged.

Callers:
- `reedcli/button.go:77` passes `buttonRunDirectory`, which returns an existing directory when it succeeds. `TestButtonRunDirectory` (`internal/reedcli/button_test.go`) calls `buttonRunDirectory` directly and never reaches `runButtonVia`, so it is unaffected.
- `buttonclick_integration_test.go:59` uses `anchorOf` returning `dir`. `dir` exists, so the new stat passes.

Tests:
- **`AnchorDirectoryThatIsGoneRunsNothingAndRaisesNoPrompt`:** `anchorGone` leaves `dir/backend` uncreated. Without the stat, a confirm entry would raise a prompt and show no message, so the row's wanted message and its `wantConfirm == nil` check both fail.
- **`ConfirmEntryWithAnUnresolvableExecutableRaisesNothing`:** it now sets `anchorMoved`, so `runDir` is `dir/backend`, created by `MkdirAll`. Reverting the message to `spec.Dir` gives `DIR/.lym/logs`, and the wanted `ANCHOR/.lym/logs` no longer matches.
- **`AnchorMoved…` rows:** `MkdirAll` makes the anchor exist, so they still pass the new stat. The gate showed 1028 passed and 0 failed.
- **Message placeholders:** the `strings.NewReplacer("DIR", dir, "ANCHOR", runDir)` is safe because neither placeholder contains the other, and `RemovedDirectory…` rows still substitute `DIR`.

Calls made:
1. `grep` for `TestButtonRunDirectory`, `RunButton(` and `anchorOf` across `*.go`: my context lacked the other callers and tests of the changed behaviour.
2. `Read` of `internal/reedcli/button_test.go`: to confirm `TestButtonRunDirectory` does not go through `runButtonVia`.
