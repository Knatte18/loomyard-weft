---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/smoke_test.go:688
    summary: malformed-status subtest reads the driver log once, the race the died-driver subtest was just fixed to avoid
    fix: In the subtest "malformed status proceeds to the success envelope and logs why", replace the single os.ReadFile of loomengine.LoomDriverLog(loc) with the same bounded poll the died-driver subtest uses (re-read every 25ms until the lowercased log contains "decode" or a 20s deadline passes), then keep the existing contains assertion. Prefer extracting that loop into one small helper in smoke_test.go that both subtests call.
---

### [LOW] Malformed-status subtest still reads the driver log once

**Location:** internal/loomcli/smoke_test.go:688

**Issue:** The `halted` seam in `internal/loomcli/start.go` reports false on a read failure, so a malformed status file keeps the handshake polling until the child exits or the run lock is seen held.
When the bootstrap sees the lock held it returns `awaitRunLockReady`, and the driver may still be between taking the lock and writing its decode error to the log.
The single `os.ReadFile` can then see an empty log, the same shape of race F2 fixed for the unrecognised-state subtest.
The window is narrow, and the Publish run passed this subtest, so it is a latent flake and not a current failure.

## Focus directive verdicts

- **Run-standalone subtest:** not a defect.
  `poisonStatusFile` sets state to `poisoned`.
  `stepLocked` in `internal/shedengine/run.go` decodes it and then refuses with `carries an invalid state "poisoned"`.
  The subtest's lowercase `invalid state` assertion matches that error, and the process runs synchronously, so the log is complete when it is read.
- **Died-driver subtest:** the read-once race was real, because `awaitRunLock` returns `awaitRunLockHalted` on its first poll for any non-running state.
  The committed poll fixes it, bounded by a 20s deadline with a 25ms sleep.
  It is deterministic in outcome, since it terminates on content or the deadline.
  Wall-clock polling is acceptable here because the subtest is tmux-tier against a real subprocess.
- **Publish failure:** `lyx gate test --tags tmux ./internal/loomcli -- -run 'TestSmokeRunStandalone|TestSmokeBootstrapLifecycle' -count=3` passes against HEAD.
  A search of `_test.go` files found no other test that still poisons a status file with an unknown field.
  The remaining "bogus" fixtures are unrelated values.
