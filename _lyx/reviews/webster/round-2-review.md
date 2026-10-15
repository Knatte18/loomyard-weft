---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedengine/follow.go:44
    summary: "set-option with the bare exact target =name is refused by tmux as no such session, so FollowRun and FollowSession never turn follow on, and every test helper that writes @lyx_follow the same way fails"
    fix: "In internal/reedengine/follow.go change the two set-option calls (FollowRun at line 44, followSessionVia at line 100) from exactSessionTarget to exactSessionWindowTarget, the =name: form that boot in lifecycle.go already uses for its set-option. Update the argv pinned in internal/reedengine/follow_test.go (the setOn closure) so the set-option target becomes =session: with the trailing colon. Change the same target in the test helpers that write @lyx_follow with the bare =name target, namely setFollow in internal/reedengine/bindings_integration_test.go:140 and the helper at internal/loomcli/smoke_drivingsurface_test.go:45. Leave the list-panes -s call on exactSessionTarget, since it is not refused. Re-run the failing Publish tests named in this review."
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/reedengine/ownwindow_integration_test.go:269
    summary: "After a server restart, Resume leaves the viewer on window @2 instead of the reopened own window @1 with follow on; the failure is deterministic and independent of F1"
    fix: "Reproduce with lyx gate test --tags tmux ./internal/reedengine -- -run TestOwnWindow_RealTmux -count=1 and find which window @2 is (likely the strands or shell window rebuilt by ensureSelvagePaneLocked or a window-zero strand relaunch in Resume, lifecycle.go line 769 onward). Then make the final viewer position of Resume the newest relaunched own-window strand while follow reads on, for example by calling selectRelevantWindowLocked once after the relaunch loop, or correct the test expectation if a later relaunch legitimately takes the viewer. The fix must match the plan card for reed follow behaviour, and the test must assert whichever behaviour that card specifies."
---

### [BLOCKING] set-option on the bare exact session target is refused by tmux

**Location:** internal/reedengine/follow.go:44 and internal/reedengine/follow.go:100
**Issue:** `exactSessionTarget` returns `=name`.
`set-option -t =name` fails on tmux 3.6 with `no such session: =name`; I reproduced it on a scratch server, where `-t =name:` and `-t name` both succeed.
`internal/reedengine/overlay.go:141` already documents that window and pane parsers need the trailing colon, and boot (`lifecycle.go`) uses it for its own `set-option`.
The consequence is that `Engine.FollowRun` logs a Warn and leaves follow off, and `FollowSession` returns an error, so `lyx reed follow` exits 1.
This is the cause of the Publish failures `TestFollow_RealTmux/*`, `TestRunCLI_ColdHub/.../FollowVerbReachesTheToldSession`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStatusStrandAcrossDriverSeeds/*`, and `TestBindings_*`, whose fixture helper writes `@lyx_follow` with the same bare target.
The unit tests (`follow_test.go`) pin the same wrong argv through a fake, so they pass while the real command cannot work.

### [BLOCKING] Resume after a server restart does not land the viewer on the reopened own window

**Location:** internal/reedengine/ownwindow_integration_test.go:269
**Issue:** `TestOwnWindow_RealTmux/ResumeAfterAServerRestartReopensDetached` fails deterministically; I ran it alone with `--tags tmux` and got `current window = "@2" after Resume, want the reopened own window "@1"`.
The follow state itself reads `on` after the restart (the preceding assertion passes), so the failure is in where Resume leaves the viewer, not in the option write, and it does not depend on F1.
I did not pin down which window `@2` is; the fix names the candidates to check.
