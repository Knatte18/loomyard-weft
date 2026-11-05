```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1 notice, F2 doc comment and F3 test row.

No findings.

- **F1, `sketchRefusedNotice`:** The new text has no `%`, so the five verbs still match the five arguments (`slug`, `refusal`, `slug`, `slug`, `slug`). The `lym board set-deps` substring survives, so the existing set-deps assertion in the notice test still holds.
- **F2, `cli.go` doc:** It is a comment only. Its order of events matches the review's account of `refuseAtStart` and `claimStart`: the guard judges first, then `claimStart` removes the run after the owner check.
- **F3, new test row:**
  - The row sets `recipe: raddle` with a status of done and makes the child-recipe path a directory.
  - `startClearsMotifRun` skips the board for a task with its own recipe, then `ReadChildRecipe` hits `os.ReadFile` on a directory. That error is not `ErrNotExist`, so it returns `false, err`.
  - `DobbyTasks` then calls `unreadableDoneRunStart.Fail` and sets `StartRefusal`, which matches `wantRefused: true`.
  - A defect that cleared the start when the read fails, such as `clears = true` on error, would leave `StartRefusal` empty and fail the row.
  - The test creates the directory with `MkdirAll` after the status file is written, so the run directory already exists and nothing collides.

Tool calls made: none. My context already held every changed file.
