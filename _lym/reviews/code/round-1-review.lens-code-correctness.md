```yaml
lens: code-correctness
outcome: reported
```

Task: check the correctness of the code the diff changes, one concrete case per branch.

**List (function or branch, case run, result)**

- `refuseOrphanedMaster` / `orphanedMasterRefusal`, Master strand, `run.lock` free: refuses with `run_ended: true`, `RunE` skipped. Correct.
- Same, Master strand with a live lock: reaches the verb. Correct.
- Same, other role or unnamed shell (`agentname.Parse` error): reaches the verb. Correct.
- Same, `RunActive` returns an error: the refusal text says "way forward: re-run the verb", but `refuseOrphanedMaster` still sets `run_ended: true`. **Wrong**, see F1.
- `heldBackReason` / `notifyLiveRuns`, `halted == ""`: nothing held back. Correct.
- Same, halted run, Webster strand: held back and listed under `notice_held_back`. Correct.
- Same, halted run, non-Webster strand (for example darn): still notified. Correct for the stated scope.
- `decisionVerb` halted derivation, `found == false`: `halted` is empty and the notice goes out. Correct.
- Same, `found` with a non-running state: `halted` is set. Correct.
- `newVerifyGate` `command` capture, dirty tree before the command renders: `command` is reset to "" at each arrival, so the report carries no command. Correct.
- Same, `verifyCommand` error: returns early, `command` stays "". Correct.
- Same, failure after the command renders: `report.Command` is set in `fail`. Correct.
- Single-opaque branch (`verifygate.go:507`), log naming no test: `Failures` set to nil and `LogTail` kept via `logTail`. Correct.
- Same, empty rerun output: `parseVerifyFailures` returns one opaque entry, so `LogTail` is "". See F2.
- `verifyGateStuckLead`, spent budget (`EndedEarly == ""`): "N of N re-prompt(s)", count and limit agree. `failedEntryReprompts` reads the failed entry's `Attempts`. The limit comes from `report.Cap`, which is `attempts` for the verify entry and `entry.Attempts` for a run-level entry. Correct.
- Same, `sent == 0` and `EndedEarly` set: the "never re-prompted" wording. Correct.
- Same, `sent > 0` and `EndedEarly` set: "N of M … ended with re-prompts left". Correct.
- Same, no entry marked failed: falls back to `gate.Attempts`. Correct for the unreadable-report path.
- `verifyGateStuckFailure`, `Entry` / `Dirty` / `Hang` / `TimedOut` / `Failures` / default branches: each ends in a way forward. The watchdog hang returns early because its own text carries one. Correct.
- `cutHead`, `cutTail`, `limit == 0`, a multibyte boundary, `len(s) <= limit`: no index overflow, and both land on a rune boundary. Correct.
- `verifyGateStuckReason`, report unreadable: counts re-prompts and ends in `wayForwardSteps(reentry)`. Correct.

**Findings**

- **F1 — MEDIUM, design.** `internal/webstercli/ownership.go:296-308` (probe-error message at `ownership.go:291`).
  - A `RunActive` probe failure is a transient read error. The refusal tells Master "way forward: re-run the verb".
  - The wrapper sets `run_ended: true` on every refusal, including that one.
  - Master's stencil (`webster-template-master.md`, new `run_ended` rung) says to end the turn with no tool call and write no file on `run_ended`.
  - A live run's Master that hits a filesystem hiccup is therefore told to quit, and the two instructions contradict each other.
  - Fix: `orphanedMasterRefusal` should also say whether the run ended, so the wrapper sets `run_ended: true` only on the lock-free case.

- **F2 — LOW, design.** `internal/websterengine/verifygate.go:507-509`, rendered at `verifygate.go:491`.
  - When the failed verify produced empty or whitespace-only output, `LogTail` is "".
  - The stuck reason then reads "…its output named no failing test, so the gate itself may have erred; output tail: ; see <log>".
  - It names a tail that is empty without saying so.
  - Fix: in the default branch, render "its output was empty" when `report.LogTail == ""`.

**Checked, not a finding**

- Whether a Master session legitimately waiting on the `waiting` rung sits in a non-running shed state (which would hold back the answer it waits for) is not visible from the diff or the reads I made. I did not raise it.

**Tool calls (4)**

1. Bash grep for `logTail`, `parseVerifyFailures`, `Cap` assignments, `FailureKindOpaque` and the `found` / `st.State` sites in `decision.go`. The manifest did not show where these are defined.
2. Read `verifygatereport.go` 145-215 for `logTail` and `wrapRunLevelEntry` (run-level `Cap` and `Attempt`).
3. Read `decision.go` 760-780 for the `readStatus` / `found` semantics behind `halted`.
4. Read `verifyparse.go` 60-125 to confirm `parseVerifyFailures` returns a single opaque entry when no failure is named.
