```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code in `landnotice.go`, `landnotice_test.go` and `dobbydeps_integration_test.go`.

**No findings.**

I ran each changed branch by hand, and each added test fails with its defect in place.

- **`flagMembers` with `pastLand`:**
  - In the stays-open row, tick 3 has the entry open, unflagged and the run still listed past Land. The new `pastLand` condition skips it, so `flags` stays `[task]`.
  - Without the condition, tick 3 would call the seam a second time, so the row fails.
- **`rememberRuns`:**
  - **Same-tick row:** `Member` is recorded at tick 1 with no pull request. At tick 2 the entry is absent and `followUp != ""`. `requested` becomes true through `note.Member`, and `ClearOwed` stays false because `carriesField` is false. The line is posted with no clear call, which matches the row. Without the `Member` branch, `sendsFor(PerTask, false)` sends nothing, so the row fails.
  - **Other cases:** With `onBoard` false the ahead-of-Land branch keeps the old `Requested` and `Member`, as before. Per-task and all-mode behaviour is unchanged when `followUp` is empty.
- **New rows:**
  - Retry row: tick 2's entry reads as flagged through `flagged[i].NotifyOnLand = true`, so `Requested` is remembered and the clear fires.
  - Non-member row: without the member filter, `wantFlags` fails.
  - Done-member row: without the done filter, `wantFlags` fails.
  - Off row: the clear is owed and no post goes out.
  - Done-entry row: the line carries the follow-up through `takeDoneEntries`.
- **Callers:** `rememberRuns` and `flagMembers` each have one call site in `landnotice.go`, both updated. `view_test.go` builds `LandNoticeState{Requested: true}`, which stays valid with the new `Member` field.
- **Seam test:** `UpdateLanding` with a no-op mutate writes a slug-only record, which `ReadLanding` reads as found with a nil `PullRequest`. That gives false, then true once the pull request is set. The lock and record directories are created up front.

Further tool calls: none. My context already held every changed file.
