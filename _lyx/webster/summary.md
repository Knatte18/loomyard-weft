# A pause holds every prompt, verify command and new batch in its scope

All seven batches (01, 05, 07, 08, 10, 13, 15) recorded `done`, none needed recovery, and no digest carried departures.
The deviations below are the digests' lists of files changed outside a batch's own targets, informational only.

- **01 pauserecord-claim-session**: `pauserecord.ClaimSession`; a cancelled `verifyrun.Run` sends SIGTERM to its group, then SIGKILL, and returns once the group is gone; `verifytree.Verify` holds before its command, stops it past the grace end and reruns after the resume; a `paused` verify marker reads as no activity and shows in loom status.
  Deviations: `cmd/lyx/daemonwakeups_test.go`, `internal/proc/procgroup_linux.go`, `internal/verifytree/hold.go`, `internal/verifytree/verify.go`, `internal/verifytree/verify_integration_test.go`.
- **05 loomshed-verify-hold**: the loomshed verify gate, the webster verify gate and `lyx webster verify` tell their pause hold.
  Deviations: `internal/shedrecipe/seam_enforcement_test.go`, `internal/testkit/envkit/envkit.go`, `internal/webstercli/verify.go`.
- **07 landing-verify-hold**: Merge-In, Finalize and Publish tell a hold that a parent lock exempts from slug entries.
  No deviations.
- **08 verified-tree-hold-pattern**: `PATTERN-verified-tree` names the hold and its tripwire checks it; shuttle's send refuses a covered target with `ErrPaused`; `PATTERN-pause-holds-prompts` added.
  Deviations: `internal/shuttleengine/fixture_test.go`, `internal/shuttleengine/pausedsend.go`, `internal/shuttleengine/pausedsend_test.go`.
- **10 shuttle-start-holds-paused-prompt**: the run start and the deferred prompt wait out a refusing pause and resend; the gate loop holds its boundary while covered; tender's and creel's runners read the hub's pause.
  Deviations: `internal/shuttleengine/gate_test.go`, `internal/shuttleengine/pausedsend.go`, `internal/shuttleengine/run_test.go`, `internal/shuttleengine/wait.go`.
- **13 paused-send-callers**: the seat notice sender retries, `lyx loom resume` and `lyx shuttle send` refuse with the way forward, and a resume line refused by another pause claims the session.
  Deviations: `internal/seatengine/engine_test.go`, `internal/seatengine/notice.go`, `internal/seatengine/seatfake_test.go`, `internal/tenderengine/resume.go`.
- **15 creel-interrupt-claims-covered**: `lyx creel interrupt` claims a covered strand, then interrupts it; begin-batch and recover-batch refuse under a pause and claim Master.
  Deviations: `internal/creelengine/seam.go`, `internal/creelengine/verbs.go`, `internal/testkit/creelfake/creelfake.go`, `internal/webstercli/batchpause_test.go`.

Final batch head: `19083529a7182ca58953600ec1e06b0f94f9df41`.

## Verify gate findings

Attempt 1 of 3 failed on one test, `github.com/Knatte18/loomyard/cmd/lyx.TestCLIForm_EveryCommand`.
The findings file noted that no recorded card touched a failing package.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `0a9f5279bcd715e463bda9bbd66f6ccdb4c2d463` fix: shorten begin-batch and recover-batch Long text under the CLI form cap
- `3a005438a5ae16f23500f2f342d4a919830efb85` fix: bring recover-batch Long text under the CLI form cap
