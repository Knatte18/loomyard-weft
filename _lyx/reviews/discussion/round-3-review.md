---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:75
    summary: D10 says the invalid-config error is raised at orch start and watcher start, but LoadConfig runs in the orch group's PersistentPreRunE, so every verb (status, cycle, stop) would refuse on a stale permission_mode, and the Decision does not say whether that is intended
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:122
    summary: D16 reuses State.Stuck for an idle-phase reason, but the field is documented as why the current phase is overdue and toIdle clears it; the Decision does not say the field's meaning widens or that the reason survives the idle tick that sets it
---

### [MEDIUM] D10 names the wrong surface for the load error

**Location:** `_lyx/discussion/decision-record.md` D10, line 75.
**Issue:** D10 says the error is raised "at orch start and at watcher start, before any pane is touched".
`internal/orchcli/cli.go` calls `orchengine.LoadConfig` in the `orch` group's `PersistentPreRunE`, for every verb except the bare group.
With the two new value checks inside `LoadConfig`, an old `orch.yaml` with `permission_mode: prompt` also fails `lyx orch status`, `cycle` and `stop`.
The operator then cannot read the state of a running watcher until the file is fixed.
The Decision gives the error's bound for start and restart but leaves open whether the other verbs refuse too.
**Fix:** State in D10 which verbs the error blocks.
Either every verb refuses with the fix named (and say so), or the value checks run only on the verbs that launch a session or watcher.
Add the matching acceptance line.

### [LOW] D16 widens the meaning of `State.Stuck` without saying so

**Location:** `_lyx/discussion/decision-record.md` D16, line 122.
**Issue:** `State.Stuck` is documented as why the current phase is overdue and waiting on the session, and the watcher clears it in `toIdle` and `enter`.
D16 sets it in phase idle for a too-short pane and clears it on the next passing probe.
The record does not say the field's meaning widens, nor how the idle-phase reason avoids being cleared by the transitions that already reset it.
**Fix:** Add one clause to D16: `Stuck` also carries an idle-phase hold reason, set and cleared only by the idle probe result, and `status` reports it.

## Focus checks

- D5, D10 and the `PATTERN-config-strictness` constraint: all three name `LoadOrTemplate` via `orchengine.LoadConfig`, which matches `internal/orchengine/config.go`.
  Absent resolves the template, present-but-invalid errors, and the value checks run in `LoadConfig` after the unmarshal.
  Not a defect apart from F1.
- D7: the Decision itself states the bound (loss of an in-flight completion, `cycle_mode: clear` restores DEFER, the compact smoke test asserts it).
  Not a defect.
- D13: the Decision states file-per-notice storage, the appender-enforced cap with transient overshoot, and at-least-once delivery across a crash between typing and removal.
  Not a defect.
