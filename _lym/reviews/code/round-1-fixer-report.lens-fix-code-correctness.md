```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code through the correctness lens, running the changed functions on concrete inputs and checking that each added test fails with its defect in place.

Findings:

- **LOW**, `internal/landingshed/prreview.go`, `syncText` and `editWarned`: `editWarned` is cleared only after a successful `Edit`.
  - Input: a transient `Edit` failure warns, the description is then reverted so the text equals `state.synced`, and a later edit fails transiently again.
  - Step by step: the revert takes the `text == state.synced` branch, which never calls `Edit`, so `editWarned` stays true and the later episode logs no Warn.
  - This is a missed warning on a rare path. Clearing `editWarned` in the `text == state.synced` case would fix it.

- **LOW**, `internal/landingshed/prreview_wait_test.go`, F5 gate coverage: no added row fails if the `!rec.ReviewEnded` check in the `wake` resync gate is dropped.
  - A `ReviewEnded` record on an open pull request ends the call at the first wake, through `judge`, before any description change can be polled.
  - The new rows do fail without the open-state check: with the closed and merged rows, `Edit` would be called at wake 7. The fixer says this itself in its notes.

Checked and sound:

- **`pollFailure` callers**: both pass a `*bool` (`&state.transientWarned`, `&state.editWarned`), and no other caller of the old signature exists.
- **`ghMutationScanPattern`**:
  - Its unquoted units exclude a raw `"`, so the scan stops at the JSON closing quote and no longer reaches the `description` field.
  - A newline is crossed only inside a quoted string.
- **`notWordPattern` with the optional quote**:
  - `"pr"` and `"view"` still fail the not-word match because the closing `\` is outside the word class, so they are not denied.
  - `"merge"` and `"release"` are denied.
- **Added tests fail with their defect in place**:
  - The persistently failing edit row logs two Warns under a shared flag, so it fails there.
  - The re-arm row logs one Warn when `textWarned` is not reset, so it fails there.
  - The zero-baseline test would see an `Edit` at its first poll if the baseline branch were removed.
- **Poll timing**: the sleeps run 1, 2, 4, 8, 16, 32, 60, 60, so the polls fall on wakes 7, 8 and 9. That matches the expected call lists and the `cancelAfter` counts.

No further tool calls made; everything was judged from the context already held.
