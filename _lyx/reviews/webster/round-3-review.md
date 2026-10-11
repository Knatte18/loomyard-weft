---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomcli/integration_driverbootstrap_test.go:56
    summary: TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver fails because its stub provider never records a SessionStart line, which card 7 now requires beside the ready pane
  - id: F2
    severity: BLOCKING
    class: scope
    location: internal/loomcli/smoke_driverstrand_test.go:75
    summary: TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps fails because writeStubDriverScript never records a SessionStart line, so startup times out as died
  - id: F3
    severity: BLOCKING
    class: scope
    location: internal/loomcli/smoke_starttail_test.go:51
    summary: TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand fails for the same stub, which writeStubDriverScript supplies, so lyx loom start exits 1 with a provider-never-ready refusal
---

## Review range

`_lyx/shed/shuttle-hook-first/status.json` gives `product.parent: main`, and `_lyx/loom/rework/` does not exist.
The range is therefore `git diff $(git merge-base main HEAD)..HEAD`, which is `c279a38ba..HEAD`, plus the Publish merge-in diff `6f7b77362^1..6f7b77362` the `## Publish failure` section names.
The merge-in brought in `planglyph`, `websterengine` and `tokencount` changes; none touches the three failing tests, so it is not their cause.

## Diff read against the cards

Cards 1 to 11 read as implemented against the plan files: the recording hooks and the `SessionStart` matcher split in `settings.go`, `SessionStartSource` and its mapping, `SessionIdleFresh` and its fold rule, `PermissionRequest` and the `notificationSignalKinds` map, the backward tail read of the fork transcript, the `landingStart` confirmation of `/clear` and `/compact`, `RecordsSessionStarts` startup gating, `sessionGone`, `StrayInputReader` and the stray clear, and the busy notice's quoted cause.
I found no defect in those beyond the one class of finding below, and I raised no comment, test-economy or per-card mechanical finding.
The three round-1 and round-2 fixes (`8aa3a2159`, `23d03a0f8`, `6d063e51f`) are in the range and read as intended.

### [BLOCKING] F1 — integration driver-bootstrap stub never records a session start

**Location:** `internal/loomcli/integration_driverbootstrap_test.go:56`, helper `integrationWriteStubDriverScript`.
**Issue:** Card 7 makes `Claude.Prepare` return `RecordsSessionStarts: true` (`claudeengine.go:186`), and `Run.checkLivenessTick` (`wait.go`, `case StartupReady`) then marks the run started only when `sessionStartRecorded` finds a session-start line in the events file.
The stub provider script prints the idle-box fixture and reads one line, but it never appends a `SessionStart` payload to the events file, which the real `--settings` hook does.
`startLLMDriverArm` therefore holds the ready pane back until the startup window ends and fails with `the provider never became ready ... no session-start line in the events file`, as `.lyx/verify/verify-4.log` shows (outcome `died`, pane showing the ready `❯` box).
Card 7 lists only shuttleengine and claudeengine files under `Edit:`, so the stubs that stand in for a Claude provider elsewhere were missed: this is a `scope` gap.
The test run in the log passes every other package, so the stubs here are the only ones the change broke.
**Fix:** Make the stub record what the real hook records.
The stub receives `--settings <path>` on its command line, and that settings file names the events path in its recording hook commands; the stub can read it from there, or the test can derive it, and append one `{"hook_event_name":"SessionStart","source":"startup"}` line before printing the idle box.
Share that behavior between this stub and F2's, since the two build the same script.

### [BLOCKING] F2 — smoke driver-strand stub never records a session start

**Location:** `internal/loomcli/smoke_driverstrand_test.go:75`, helper `writeStubDriverScript`.
**Issue:** The same cause as F1: the script prints the idle box, reads one line, prints it again and sleeps, and writes no session-start line.
`TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` runs `lyx loom start`, which exits 1 with `the provider never became ready` (`verify-4.log`, `smoke_driverstrand_test.go:190`).
**Fix:** The same as F1: the stub appends a `SessionStart` line to the events file.
This test's `driverShuttleConfig` lowers `startup_timeout_s` to 10, so the failure surfaces after that window.

### [BLOCKING] F3 — smoke start-tail test uses the same stub

**Location:** `internal/loomcli/smoke_starttail_test.go:51`, which takes its provider from `writeStubDriverScript`.
**Issue:** `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` exits 1 for the same missing session-start line (`verify-4.log`, `smoke_starttail_test.go:78`).
No separate defect exists in the test; fixing F2's helper fixes it.
**Fix:** The F2 fix covers it.
Re-run all three tests under `lyx gate test --tags tmux ./internal/loomcli` (or the tag the file carries) after the stubs change, and look for any other fake provider that launches through shuttle with the `claude` config key pointed at a script, such as the `internal/orchcli` smoke files, in case they reach the same startup path.
