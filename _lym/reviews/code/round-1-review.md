---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    origin: handler
    location: internal/webstercli/reset.go:193
    summary: On a partial records commit, the reset's records commit lands but the verb refuses before its synchronous push, so an owed push of that commit writes no record and does not hold the run, contrary to the change description's claim that the verb pushes once the records commit is made.
    fix: "Leave the code in place and defer this finding to the run's parent. The record's `No bug fixed` entry rules that nothing changes, and the review range is empty, so a ruling on this edge belongs to the parent. If the parent wants it closed, it files a board note for a new task: when `fabricSync` returns an error after the weft commit landed, the reset runs `c.pushRecords` before it refuses."
---

The review range `3fb0787e3..HEAD` holds no commits and no hunks, since HEAD is the review base.
The amended board entry's `No bug fixed` entry asks for no change, so the empty diff matches its scope.
This round checked the change description's claims about the base against the code at HEAD.

### [LOW] A partial records commit skips the reset's synchronous push

**Location:** `internal/webstercli/reset.go:193`

**Issue:** `fabricSync` returns the error of `Fabric.CommitWithTrailers` even when the weft commit landed (`internal/webstercli/sync.go:38-41`).
`commitBothSides` maps a weft commit that landed but failed to record onto a `*PartialCommitError` (`internal/fabricengine/commit.go:242-245`).
`commit` then spawns the detached pusher and returns that error (`internal/fabricengine/commit.go:169-176`).
On that path `reset.go:192-194` refuses with `fabricSyncWayForward` before the synchronous push at `reset.go:202-211` runs.
The commit carrying `Run-Transition` is left to the background pusher, which writes no owed record (`pattern/PATTERN-pair-remote.md:29`), so an unreachable origin does not hold the run.
This is the gap the board entry reported, narrowed to one rare failure path.
The change description says the verb pushes "once the reset's records commit is made", and that claim does not hold on this path.
The finding is a `decision` because the parent's `No bug fixed` entry rules that nothing changes, and only the parent can reopen that ruling.

## Rejected

- The claim that a takeover by another machine fails the verb holds: `PushRun` returns the hand-over's `ErrHandedOver` (`internal/pairteardown/handover.go:128-130`), and `reset.go:204-205` fails the verb on it. A hand-over that itself fails is reported in `push_error`. That still leaves a failed-push record and is no defect.
- The claim about `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` holds: its `UnreachableRecordsOriginLeavesAnOwedRecord` row asserts both `push_error` and `FailedPushesOwedBy` (`internal/webstercli/reset_test.go:210-216`).
- Both pattern documents describe the reset's synchronous records push and its owed record as the code behaves (`pattern/PATTERN-pair-remote.md:40-41`, `pattern/PATTERN-run-fsm.md:32-35,46`).
- No lens raised a finding, because the diff is empty.
