---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 reads each audited session through its shuttle run record, but that record is deleted on a done run and webster's state names no run record for Master's sessions or for earlier recovery sessions, so the per-run engine has nothing to read and falls back to the default engine
    fix: In D6, replace the read from the shuttle run record with a webster-owned record. At spawn, webster writes the engine name the spawn resolved to (the run's RunState or Result engine) into its own state beside each session id it records, that is beside MasterSessionID and each batch's SessionID for Master, and beside each RecoverySessions entry for the recovery strand. The lookup reads that state entry. A state entry with no engine name, written before this change, reads as the current default, and that case keeps D6's existing refusal row and its shuttle.yaml way forward. Add a Scope In line for the new webster state key, a Constraints line that it is additive under PATTERN-tolerant-records, and an acceptance criterion that reset and a later recover-batch read a recovery session whose shuttle run dir was removed after its done run through that session's own recorded engine.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 places the wiring check before Master spawns but does not say which verbs run it, so either recover-batch spawns a recovery strand on a non-audit engine after a mid-run recovery key change, which makes the internal-error claim false, or every verb checks both keys and a master key change blocks record-batch and reset
    fix: In D6, state that each verb's wiring checks only the role engines that verb spawns. run checks master and recovery before Master spawns. recover-batch checks recovery before it spawns a recovery strand, naming recovery. Verbs that spawn nothing, such as record-batch and reset, check no key and depend only on the lookup. Keep the sentence that a recorded engine value without fork audit is an internal error, since this placement makes it unreachable. Add an acceptance criterion that recover-batch with the recovery key changed mid-run to an engine without fork audit is refused before any pane exists, naming recovery.
---

### [BLOCKING] D6's recorded engine has no record to read once a run is done

**Location:** `_lyx/discussion/decision-record.md`, D6.

**Issue:** D6 says `websterengine` reads each run's engine from that run's own record: Master through Master's record, and the recovery strand through its own record.
`loadRunWrites` (`internal/websterengine/runwrites.go:29`) audits every session id `recordedSessions` lists, and `reset` and `recover-batch` both read it.
Webster's state records only session ids for these sessions: `MasterSessionID` (`state.go:150`), each batch's `SessionID` (`state.go:237`) and `RecoverySessions` (`state.go:295`).
The only run directory it records is `ShuttleRunDir` (`state.go:283`), which holds just the latest recovery spawn of one batch.
The run directories don't survive either.
Shuttle removes a run's directory on a done outcome (`internal/shuttleengine/wait.go:1689-1699`).
`recover-batch` removes the recovery's `ShuttleRunDir` once its digest is done (`recoverbatch.go:792-794`).
The orphan sweep removes any run directory whose strand is gone (`rundir.go`, the sweep near line 335).
So a later `reset`, or a `recover-batch` for a later batch, audits an earlier recovery session, or a Master session from an earlier `run`, whose run record no longer exists.
Under D6's own rule, an absent engine value then reads as the current default.
That is the wrong-provider reading D6's rationale sets out to remove, and it happens with no refusal.
D6's acceptance criterion on a mid-run `master` key change only covers the live run, so it does not catch this case.
The engine has to be recorded where the session id is recorded, in webster's state.

### [MEDIUM] D6 does not say which verbs run the wiring check

**Location:** `_lyx/discussion/decision-record.md`, D6.

**Issue:** Each webster verb is wired on its own: `webstercli/wiring.go:157` and `:296` resolve the roles and `:202` and `:306` resolve the audit engine for each invocation.
`recover-batch` spawns the recovery strand from `deps.Roles[RoleRecovery]` (`recoverbatch.go:300`), which is resolved from `webster.yaml` when that verb runs, not when Master spawned.
D6 says only that the wiring checks both role engines "before Master spawns".
If that check runs only in `run`, a `recovery` key changed mid-run to an alias whose engine has no fork audit passes unchecked.
`recover-batch` then spawns on that engine, and its recorded engine reaches the lookup.
That contradicts D6's claim that such a recorded value is unreachable and an internal error.
If instead every verb's wiring checks both keys, a mid-run `master` key change refuses `record-batch` and `reset` by key.
Yet D6 says those verbs read Master through its recorded engine, which the key change does not affect.
Either reading breaks one of D6's own sentences, so D6 needs to name the verbs that run the check.

## Focus verdicts

- D6, fork-audit checks per role: both role engines are checked and each refusal names its own key, but the check's verb placement is open (F2).
  The Bound covers the relaxed default-engine check.
  It does not cover the fact that the per-run lookup falls back to the default whenever the run record is gone (F1).
- D6, way forward per check point: the wiring refusal's way forward (set the role's key) clears it.
  The lookup refusal for an engine-less record names `engine:` in `shuttle.yaml`, which changes the default it reads as, so that way forward clears it too.
  The refusal can be reached far more often than D6 assumes, because records are deleted (F1).
- D8, `lyx burler run` with a cluster fan: `internal/burlercli/run.go:182` builds the reviewer's choice from `--review-model` with no engine.
  D8 names the default engine and the `shuttle.yaml` way forward for that caller, and an acceptance criterion covers it.
  Not a defect.
- D2's scan: the rule forbids any read of a `modelspec.Resolved` field outside `modelspec` and `NewModelChoice`, and the acceptance criterion requires the scan to fail on a new read of any of the three fields.
  That catches a site that copies `Model` and drops `Engine`.
  Not a defect.
- D5 against Scope's Out rule: in this repo's `contracts/stencils`, only the seat directives message a session other than the parent.
  The driver's notify stencils message the parent.
  Discussion-Write's delivery prompt goes to `parent.Name` (`internal/loomcli/wiring.go:416`).
  All the parent-messaging roles are classified alike, without `peer-messaging`.
  Not a defect.
