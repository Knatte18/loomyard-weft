```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Lens fix-consistency: check that the F1–F3 change set agrees with its siblings and twins.

Checked:
- **Doc comment (`buttonclick.go:67`):** it now lists "an anchor directory that is gone". That matches the new `os.Stat(runDir)` branch, which says one message and runs nothing further.
- **Existing-check message:** the new message "the anchor directory … is gone" has the same shape as the `spec.Dir` "is gone" message. Each is a single `say` followed by `return`, so neither raises a prompt or runs anything.
- **Integration test:** `buttonclick_integration_test.go:59` uses `anchorOf` returning `dir`, which is `spec.Dir` and already passes the first stat. The new check does not affect it.
- **`TestButtonRunDirectory` (`internal/reedcli/button_test.go`):** it tests `buttonRunDirectory` and creates `backend` on disk. The new check lives only in `runButtonVia`, so this test is unaffected.
- **Unit-test rows:**
  - The `removeDir` rows stop at the first stat, before `anchorOf` runs.
  - The `anchorMoved` rows create `dir/backend` on disk, so they pass the new check.
  - The `anchorGone` row leaves it uncreated, so the check fires with `confirming` and no prompt.
  - Rows with `anchorErr` set return before the new stat.
  - The `ANCHOR` and `DIR` placeholders cannot collide: `DIR` is not a substring of `ANCHOR`, and the replacer works on `tt.wantMessage`.
- **F2 row:** the `exeErr` row now has `anchorMoved: true` and wants `ANCHOR/.lym/logs`. The unmoved prompt-failure case is no longer covered, but it is the same behavior, so that is not a defect.
- **Info log (F3):** the reworded message no longer claims the anchor changed. It has no remaining twin, since the grep for the old phrase finds nothing outside `.lym` and `_lym`.
- **Header comment (`buttonclick.go:1-2`):** it already says the run directory comes from the told anchor reader, so it needs no edit.

Further tool calls:
1. `grep` for `anchorOf`, the old Info text, `TestButtonRunDirectory` and the prompt-failure message: my context did not show twins in files outside the two that changed.
2. `Read` of `internal/reedcli/button_test.go`: to check whether the anchor-directory test there depends on the changed behavior.
