---
round: 2
focus:
  - "Where edit records are written and read: `internal/shuttleengine/claudeengine/editrecord.go` (`RecordEdit`), the edit hook command in `internal/shuttleengine/claudeengine/settings.go`, and `watchEditLoops` in `internal/shuttleengine/editloop.go`. Settle whether any edit-loop write can still share a file with the recording hooks' two-step payload-and-newline appends, and whether the reader's cursor and `RunState.EditLoopNotifiedOffset` refer to the file the writer uses."
  - "Failure paths in `watchEditLoops` in `internal/shuttleengine/editloop.go`: settle whether a failed `persistEditLoopNotifiedOffset` or a notifier error leaves the loop's counts and the cursor in a state where a later tick retries the notice, as `notifyDialog` does."
  - "Tests in `internal/shuttleengine/editloop_test.go` (`TestWait_EditLoopNotice`) and `internal/shuttleengine/claudeengine/editrecord_test.go` (`TestRecordEdit`): settle whether a regression that ignores the hook-side stamp time, drops a field from the notice line, or swallows a `RecordEdit` open failure would fail a test."
  - "Docs in `docs/overview.md`, around the autonomous run's parent-notice list and the PostToolUse hook batching sentence: settle whether the edit-loop notice, the hidden `hook-edit` verb and the `edit_loop_edits` and `edit_loop_window_s` keys are named."
---

The focus lists places to check after the round-1 fixes; it caps no severity and states no verdict.
