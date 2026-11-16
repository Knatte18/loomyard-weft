```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the merge 4153c5ef and c1a136c7 add or change, find the test row that would fail if it broke.

**Behaviours and the tests that pin them:**

1. **A resumed orch session gets remote control (merge settlement in `start.go`).** `launchOrch` now calls `resumeOrch(st.SessionID, remoteControlName)`, and `resumeOrch` passes the name to `orchSpec`.
   - Pinned by `TestStart_LaunchPassesAndRecordsRemoteControl/ResumedSession` (`internal/orchcli/remotecontrol_test.go:48`). It asserts `spec.RemoteControlName == "neve::tst:orch"` together with `spec.ResumeSessionID == "sess-1"` (`remotecontrol_test.go:75-79`).
   - If `resumeOrch` dropped the name, this row would fail.
2. **The retry over a live holder keeps remote control.** The beam's `resumeOrch` retries with the same `spec` (`start.go:766-776`).
   - `TestStart_ResumeOverALiveHolder` (`internal/orchcli/start_test.go:1610`) pins that the retry resumes the recorded session (`start_test.go:1671`). It sets no remote-control switch.
   - The retry reuses the one `spec` variable that row 1 pins, so a regression here would need a new spec built for the retry. Coverage is indirect, and I file no finding.
3. **Fresh launch with remote control (`start.go:752`).** The merge left this line unchanged on main's side. `TestStart_LaunchPassesAndRecordsRemoteControl/SwitchOn` pins it.
4. **`endTestMap` and its wiring are dropped (`cli.go`, settled to main's side).** No test is needed, because the behaviour is gone. Main's own `stop.go` change removed the bring-down step. `TestBeamDown_StopsOrchThenTheDobbyByRole` comes from main and is not this lens's subject.
5. **`RoleTestMap` is removed from `subjectPassSpared` (`internal/runproc/end.go:100`).** The roles still spared (`RolePush`, `RoleSlackRelay`, `RoleTeardown`, `RoleAdopt`, `RoleSync`) are still pinned by `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`.
   - The `TheHubOwnerSubject…` row expects only `term pid 84` and `term pid 85` (`end_test.go:524-528`).
   - The `"beam"` rows still pin that adopt and sync are spared, along with the ordering: runs, then the subject, then its dobby last (`end_test.go:503-521`).
   - Removing the `PID 81 RoleTestMap` fixture record drops nothing that a row asserted.
6. **Doc-comment changes (`runproc/doc.go`, the `end.go` doc, and the `resumeOrch` doc in c1a136c7).** These are static prose, so they are exempt.

**Findings:**

- **LOW, consistency.** `internal/runproc/end_test.go:524`: the subtest name `TheHubOwnerSubjectLeavesTheHubWidePushesTestMapSlackRelayAndTeardownAlone` still says "TestMap". The merge removed the `RoleTestMap` record from the fixture (`end_test.go:479-485`, now pushes, Slack relay, teardown, orch and dobby only), so the name now claims coverage the row no longer has. Evidence: `end_test.go:479-485` and `:524`.

Tool calls:
1. grep of `internal/orchcli/start_test.go` and `remotecontrol_test.go` for resume, remote-control and `SessionHeldError` test rows. The manifest held no orchcli test files.
