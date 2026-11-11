```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set through the fix-consistency lens, meaning whether the changed text agrees with its siblings and twins.

**Findings**

- LOW: `internal/dobbyengine/landnotice.go`, file header, lines 5–6.
  - The header says "While a beam's pull request is recorded, the beam's members ask for it. Their landed line then carries the beam's follow-up."
  - After the fix, a member remembered ahead of Land also gets the line under per-task with no flag set. This holds for a member whose flag was never written and for one whose entry is already done.
  - The `landNotices` and `rememberRuns` docs state this rule. The header still describes only the flagging path, so it is a little stale.
  - Evidence: the new `followUp != "" && (note.Member || ...)` branch in `rememberRuns`, and the `AMemberLandingInTheTickThePullRequestIsRecordedGetsTheLine` and `ADoneMemberEntryIsNotFlagged` rows, which expect a line with `wantFlags` empty.

**Checked, consistent**

- `LandNoticeState.Member` is a new field with `omitempty`. The only other users of the type are `state.go:44` and `view_test.go:162`, and neither depends on the field set.
- `ClearOwed = carriesField` matches the old `ClearOwed = requested` whenever followUp is empty. No prime or no-pull-request path changes.
- Both callers of `flagMembers` and `rememberRuns` are updated: `landNotices` is the only one, and the tests call it through `landNotices`.
- The `takeDoneEntries` doc still holds, because the run is decided in `rememberRuns` before it is considered.
- The test harness switched from `len(flags) > 0` to `flagged`. Every older beam row then behaves as before, since `failFlags` defaults to 0.

Tool calls made:
1. `grep` for `clear_owed|ClearOwed|LandNoticeState` outside `landnotice*.go`, to find twin schema or docs. My context lacked a repo-wide search.
