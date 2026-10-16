# Round 2 fixer-report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise confirmed: `loadRunWrites` (`internal/websterengine/runwrites.go`) audits every recorded session id, the state holds only ids (`MasterSessionID`, each batch's `SessionID`, `RecoverySessions`), shuttle removes a done run's directory (`internal/shuttleengine/wait.go`), and `recover-batch` removes a done recovery's `ShuttleRunDir`.
  D6 now records each session's engine in webster's state beside its id: `masterEngine` from the run's state at Master's spawn (where `runlevel.go` captures `MasterSessionID`), each batch's `engine` copied at begin-batch like `SessionID`, and `recoveryEngines`, a map from session id to engine, at each recovery spawn.
  The lookup reads those entries.
  An engine-less entry from before this change reads as the current default and keeps the existing refusal row and its `shuttle.yaml` way forward.
  Added a Scope In line for the state keys, extended the `PATTERN-tolerant-records` constraint to cover them, and added the acceptance criterion for `reset` and a later `recover-batch` reading a recovery session whose run directory was removed.
- F2 (MEDIUM), fixed as proposed.
  Premise confirmed: `internal/webstercli/wiring.go` resolves roles and the audit engine for each verb invocation, and `recover-batch` spawns from `deps.Roles[RoleRecovery]`.
  D6 now says each verb's wiring checks only the role engines it spawns: `run` checks master and recovery before Master spawns, `recover-batch` checks recovery before it spawns a recovery strand, and `record-batch` and `reset` check no key.
  The internal-error sentence stays.
  The Bound is reworded to match.
  Added the acceptance criterion for `recover-batch` after a mid-run `recovery` key change.
- Logged both under a Round 2 heading in `support-log.md`.

## Disputed

None.

## Deferred

None.
