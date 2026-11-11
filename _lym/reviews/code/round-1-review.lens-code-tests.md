```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. I list each behaviour the diff adds or changes, the test row that pins it, and a finding for each behaviour with none. I made no tool calls and judged from the inherited context.

**Behaviour → pinning test**
- `SetNotifyOnLand` sets the field: `TestNotifyOnLand` row "set sets it" (`internal/boardengine/board_test.go`).
- `SetNotifyOnLand` writes nothing for an entry already carrying it, or for an absent slug: `TestNotifyOnLand` rows "set of an entry carrying it writes nothing" and "set of an absent slug writes nothing".
- `hubDobbyDeps` wires `SetNotifyOnLand` to the hub board: `TestHubDobbyDeps_NotifyOnLandSeamsWriteTheHubBoardEntry`.
- `hubDobbyDeps` wires `PullRequestRecorded` to read the landing record (`found && record.PullRequest != nil`): none.
- A beam with a recorded pull request flags an open member and its landed line carries the follow-up: `TestLandNotices` row `APullRequestFlagsAMemberAndItsLineCarriesTheFollowUp`.
- A member already flagged by hand gets one line and no second flag: row `AMemberFlaggedByHandGetsTheSameOneLine`, which asserts `wantFlags` is nil.
- No recorded pull request means no flag and no follow-up: row `ABeamWithNoPullRequestFlagsNothing`.
- An unlanded line carries no follow-up: row `AnUnlandedMemberLineCarriesNoFollowUp`.
- A failed landing-record read decides nothing until it reads: row `AFailedLandingRecordReadDecidesNothingUntilItReads`. A line posted without the follow-up on the failed tick would break its `want`.
- The prime's dobby and a nil `PullRequestRecorded` flag nothing: the non-beam rows, whose `SetNotifyOnLand` stub must stay uncalled (`wantFlags` nil).
- The `flagMembers` filter on project, kind and done status: partly pinned. Only the positive case (member of `b`) and the already-flagged case have rows. Non-member, done member and non-task/project kind: none.
- A failed `SetNotifyOnLand` write is logged and retried next tick: none. The stub always returns nil.
- The follow-up on the "landed, commit unknown" line: none. Only the `landed as <sha>` form has a follow-up row.
- The follow-up on a done entry taken up by `takeDoneEntries` (the `followUp` argument): none. The `doneEntryParent` rows all have `beam` false.
- `land_notice: off` with a recorded pull request sends no line but still flags: none.
- The removed geometry `AnnouncePath`: `TestDobbyGeometry` updated.
- The `PullRequest` record without `AnnounceHead`: `TestLandingRecord`, `TestPROpen`, `TestPRUpdate` and `TestMoveOffRetired` updated.
- An old `announce_head` key is ignored on read: no dedicated test. It is covered only by the repo-wide `DisallowUnknownFields` guard, which `PATTERN-tolerant-records` names.

**Findings**

1. MEDIUM, design. `internal/beamcli/dobbydeps.go:238-241`. The new `deps.PullRequestRecorded` closure has no test.
   - It decides whether a beam's dobby flags members and adds the follow-up.
   - `found && record.PullRequest != nil` is untested for an absent record, a record with no pull request, and a recorded pull request.
   - Evidence: `dobbydeps_integration_test.go:539-560` drives only `SetNotifyOnLand` and `ClearNotifyOnLand`.
   - Fix: add rows to that test, or to the `hubDobbyDeps` seam test, that write a landing record and assert the seam's answer.

2. LOW, design. `internal/dobbyengine/landnotice.go:158-168` (`takeDoneEntries`, `landNoticeLine`). The follow-up on a done entry's line, and on the `landed, commit unknown` line, is not pinned.
   - Evidence: the `beam: true` rows in `landnotice_test.go` all use a running-then-landed run.
   - Fix: add a `TestLandNotices` row with `beam`, `pullRequest` and `doneEntryParent`.

3. LOW, design. `internal/dobbyengine/landnotice.go:178-196` (`flagMembers`). The member filter (other project, done status, kind) and the failed-write retry are not pinned.
   - Fix: add rows to `TestLandNotices`, such as a non-member and a done member that stay unflagged. For the failure, make the `SetNotifyOnLand` stub fail once, then expect a flag on the next tick.

4. LOW, design. `internal/dobbyengine/landnotice.go` (`land_notice: off` with a recorded pull request). No row pins that no line goes out and the follow-up goes with it, although `summary.md` states this as a design decision.
   - Fix: add a `TestLandNotices` row with `LandNoticeOff`, `beam` and `pullRequest`.

No duplicate or redundant new tests found. The redundancy report judged none of them.

Tool calls: none.
