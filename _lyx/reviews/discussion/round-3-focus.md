---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md` D6, settle where a session's engine is recorded and read: shuttle deletes a run's directory on a done outcome, `recover-batch` removes a recovery's `ShuttleRunDir` once its digest is done, and webster's state (`internal/websterengine/state.go`) records only session ids (`MasterSessionID`, each batch's `SessionID`, `RecoverySessions`). Does every session that `loadRunWrites` (`internal/websterengine/runwrites.go`) audits for `reset` and `recover-batch` have an engine record that survives, and does an absent engine value never silently read as the current default engine?"
  - "In D6, settle which webster verbs run the fork-audit wiring check: `webstercli/wiring.go` resolves roles and the audit engine per invocation, and `recover-batch` spawns the recovery strand from the role resolved when that verb runs. Does each verb check exactly the role engines it spawns, and does every sentence in D6 about an unreachable or refused recorded engine hold under that placement?"
  - "For every new state key, verb, refusal or escape hatch D6 introduces, does the introducing Decision state what it can skip or let through and what bounds it, and does a way forward and an acceptance criterion cover each refusal it adds?"
---

Focus targets the D6 revisions the round-2 fixes will make; the other Decisions were not faulted in round 2.
