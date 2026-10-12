---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/smoke_test.go:366
    summary: TestSmokeRunStandalone failure_before_the_first_persist asserts the driver log names a decode failure, but the poisoned-state rig now produces an invalid state refusal
    fix: In the "failure before the first persist leaves a non-empty driver log" subtest, change the check at internal/loomcli/smoke_test.go:366-368 from strings.Contains(strings.ToLower(string(content)), "decode") to strings.Contains(strings.ToLower(string(content)), "invalid state"), and change its message to "want it to name the invalid state". This matches the shedengine error "carries an invalid state" that poisonStatusFile now provokes, and matches the assertion already used in the died-driver subtest.
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/smoke_test.go:616
    summary: The died-driver bootstrap subtest races the driver because a status with state "poisoned" makes the start handshake report halted on its first poll, before the driver has written its log
    fix: In the "died driver proceeds to the success envelope and logs why" subtest, after the successful runLoomCLINoFatal call and before the single os.ReadFile of loomengine.LoomDriverLog(loc), poll that log until it contains "invalid state" or a deadline of about 20 seconds with a 25ms sleep passes, as waitForCurrentProducer does, and then run the existing assertion on the last content read. The start handshake (awaitRunLock in internal/loomcli/bootstrap.go) returns awaitRunLockHalted on its first poll for any status whose state is not running, so bootstrap exits while the spawned driver is still starting. Do not change poisonStatusFile to a state of running, because the driver would then not fail at the read gate.
---

### [BLOCKING] Publish failure: driver-log assertion in TestSmokeRunStandalone is stale

**Location:** internal/loomcli/smoke_test.go:366
**Issue:** The failing run (`.lyx/verify/verify-7.log`) shows the driver log as `{"error":"shedengine: status file ... carries an invalid state \"poisoned\"", ...}` while the subtest wants the lowercased log to contain "decode".
Card 2 made `state.ReadJSONNoMkdir` tolerate unknown fields, so the old unknown-field rig no longer fails the read gate.
The branch rewrote `poisonStatusFile` to set `state` to `"poisoned"`, and updated the died-driver subtest's assertion to "invalid state", but left this subtest's "decode" assertion unchanged.
The refusal now comes from `Shed.stepLocked`'s `st.State.valid()` check in `internal/shedengine/run.go`, whose message never says "decode".
The subtest is a smoke test and cannot pass until its assertion matches the rig.

### [BLOCKING] Publish failure: died-driver subtest reads the driver log before the driver writes it

**Location:** internal/loomcli/smoke_test.go:616
**Issue:** The failing run shows `driver log = ""` at `smoke_test.go:639`.
The cause is the rig change, not the driver.
Before this branch, `poisonStatusFile` added an unknown field.
The bootstrap handshake's `halted()` closure (`internal/loomcli/start.go`) read the status through the strict reader, got a decode error, and reported false, so the handshake kept polling until the driver died, by which point the driver had logged its error.
With `state: "poisoned"` the status decodes, `st.State != shedengine.StateRunning` holds immediately, and `awaitRunLock` (`internal/loomcli/bootstrap.go`) returns `awaitRunLockHalted` on the first poll: the lock is not yet held, the freshly spawned child is alive, and `halted()` is true.
`lyx loom start` then prints the success envelope while the driver is still starting, and the test's single `os.ReadFile` of the driver log sees it empty.
The behavior under test, that the bootstrap proceeds and the reason survives in the log, still holds; the test's one-shot read is what is wrong.
The fix polls for the log content rather than changing the rig, so the invalid-state refusal keeps exercising the read gate.
