```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `_lym/landing/summary.md` edits for agreement with the rest of the summary and the diff.

Findings:

1. **MEDIUM** — "14 files left with no test are deleted" (Other test-code changes).
   - The fix commits re-added `internal/fabricengine/remove_siblingdirty_integration_test.go`.
   - `git diff --diff-filter=D --name-only f2ac2bf7 HEAD` now lists 13 deleted files; at `feaa74d0` it listed 14.
   - The sentence still says 14. A restored test file makes it wrong.
   - The bullet also pins a perishable tally. A derived phrase like "files left with no test are deleted" would avoid the stale number.

2. **MEDIUM** — the timing table, "After the last commit" column (Numbers section).
   - It still reads Passed 1366, wall time 44.2 s and package time 34.2 s. Those were measured at `feaa74d0`.
   - Ten tests were restored afterwards, in `9e969b84` and `b8308134`, so "After the last commit" no longer describes HEAD.
   - The prose under the table ("The package time fell by 1.7 s…") rests on those numbers.
   - Either re-measure, or relabel the column as measured at `feaa74d0`.

3. **LOW** — the new text in the "Dropped" bullet.
   - It reads "Their standalone cost in the study is no longer totalled here, since ten of the originally dropped tests came back."
   - It drops the 51.5 s figure without replacing it. The prose beneath the table still says most dropped tests cost 0.2 to 0.5 s standalone.
   - The wording is also awkward: "no longer totalled" reads as editing history.

4. **LOW** — the restored-tests list.
   - The summary's Numbers section says the 67 pre-marked keeps and 54 newly kept tests sum to 121. The new "Restored after the first review" subsection sits under "What was kept, and why" but is not in that tally.
   - The arithmetic holds (252 = 188 + 64), but the review states the delta as 11 (65). The summary now says 10 (64), and the one-line difference is not explained. The 65th added line is the reworded `TestCwdMutation_MigratedFilesStayChdirFree` keep line. This is probably correct, but the reader has no note of it.

Checked and consistent:
- 188 + 64 = 252.
- The ten restored test names match the keep lines in the diff.
- `export_test.go` is unchanged since `feaa74d0`, so the "shims deleted" bullet still holds.

Tool calls: 1. `git diff --diff-filter=D/A --name-only` and `--stat` between `f2ac2bf7`, `feaa74d0` and `HEAD`. My context lacked the deleted-file counts and the files the fix commits added.
