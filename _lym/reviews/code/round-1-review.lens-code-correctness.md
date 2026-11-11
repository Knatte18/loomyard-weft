```yaml
lens: code-correctness
outcome: reported
```

**Task restated:** code-correctness lens on the one-land-notice diff. Each function or branch the diff changes is listed below with the case I ran.

## Per-function list

- **`pullRequestFollowUp` (`landnotice.go`)**
  - Case: nil seam or empty slug. It returns `"", true`. Correct.
  - Case: read error. It returns `"", false`. Correct.
  - Case: not recorded. It returns `"", true`. Correct.
  - Case: recorded. It returns the follow-up text with the slug. Correct.
- **`flagMembers`**
  - Case: nil seam. It returns `tasks` unchanged. Correct.
  - Case: member that is open and unflagged. It is flagged in the clone only on success. Correct.
  - Case: seam error for one member. The remaining members are still tried, the failure is logged once, and the member's flag stays false in the clone. Correct.
  - Case: done member, member that is already flagged, or an entry whose project differs. It is skipped. Correct.
  - Case: re-flagging an open entry whose flag the dobby just cleared after landing. **Wrong** (finding 1).
- **`landNotices`**
  - Case: board read fails. Flagging is skipped and the request refresh is skipped. Correct.
  - Case: landing-record read fails. `decides` is false, so no line is decided and `takeDoneEntries` is skipped. This matches the old `beam.yaml` failure handling. Correct.
  - Case: prime dobby (`Slug == ""`). The follow-up is empty and nothing is flagged. Correct.
- **`rememberRuns`**
  - Case: run ahead of Land. It takes the flagged request, because the cloned `tasks` feed `requests`. Correct.
  - Case: run past Land with a remembered request. The line is built with `followUp` and `ClearOwed` is set. Correct.
  - Case: `decides` false. It skips the run and keeps the memory. Correct.
- **`landNoticeLine`**
  - Case: landed with a commit. The follow-up is appended. Correct.
  - Case: landed with no commit. The follow-up is appended. Correct.
  - Case: not landed. No follow-up. Correct.
- **`Board.SetNotifyOnLand`**
  - Case: absent slug, or the flag already set. It returns `noWrite{}`, so nothing is written. Correct.
  - Case: flag unset. It sets the flag and writes. Correct.
  - Case: kind `note`. It is not checked here. The dobby only passes task and project members, so this is correct.
- **`hubDobbyDeps` seams**
  - Case: `PullRequestRecorded` with no record. `found` is false, so it returns `false, nil`. Correct.
  - Case: `PullRequestRecorded` on a read error. `found` is false, so it returns `false, err`. Correct.
  - Case: `SetNotifyOnLand`. `OpenHub` errors are propagated. Correct.
- **`carryPullRequest`, `prOpen` and the removal of `AnnounceHead`**
  - Case: a legacy record that still has an `announce_head` key. `state.ReadJSON` is tolerant per PATTERN-tolerant-records. Correct.
- **`tick`**
  - Case: `landNotices` runs before `startReady` (`loop.go:669` against `loop.go:671`). A member filed while the pull request is recorded is flagged before it starts. Correct.

## Findings

1. **MEDIUM, class `design`, `internal/dobbyengine/landnotice.go` (`flagMembers`, `settleLandNotices`; `PastLand` includes the teardown row per `battenrecipe/land.go:25`):** a possible second notice for one landing.
   - A member's run is past Land but its board entry is not yet `done`; the "AllTeardownRowCountsAsPastLand" row suggests this state exists.
   - The notice is posted and the flag is cleared. The memory is dropped.
   - On the next tick `flagMembers` sees an open entry that lacks the flag and sets it again.
   - The entry then reaches `done` carrying `notify_on_land`, and `takeDoneEntries` finds no memory and posts a second landed line.
   - This breaks "exactly one notice per task" for a beam with a recorded pull request.
   - I could not confirm when the entry's status becomes `done` relative to `PastLand`, so the window's width is unverified.
   - The tests don't cover it: the harness sets `h.fake.tasks` only while a run is ahead of Land (`landnotice_test.go`, the `PastLand` case in the loop).
   - Suggested fix: skip flagging any member that has a listed run past Land, or that has landing-notice memory.

2. **LOW, class `design`, `landnotice.go` (`flagMembers`):** the flag outlives the recorded pull request.
   - If the landing record's pull request is dropped, members flagged by the dobby keep `notify_on_land`.
   - Under `land_notice: per-task` they still send a line when they land, without the follow-up.
   - A dobby-set flag cannot be told apart from a hand-set one, so nothing unsets it.
   - Not clearly wrong against the board entry, which is silent on this. Mention only.

3. **LOW, class `design`, `landnotice.go` (`flagMembers`):** it flags members with `OwnedElsewhere` set (or whose parent is another worktree) and writes their board entries.
   - `takeDoneEntries` skips those same entries, so no line comes from this machine.
   - The write is idempotent, so the effect is harmless.
   - Only note it, because the owner-machine rule governs starting and landing, not flagging.

## Tool calls

1. `grep` for `PastLand`, `IsDoneStatus` and done-status writers. I needed to know when the board entry becomes done relative to `PastLand`; my context lacked it, and the result only confirmed that `PastLand` is the batten-run state check.
