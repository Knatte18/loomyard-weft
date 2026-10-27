---
round: 2
exclude_lenses: []
focus:
  - "Decision 3, Decision 5 and the Acceptance criteria of decision-record.md: when runDobby (internal/beamcli/cli.go) exits before dobbyengine Run, through a placeDobby refusal or a hubDobbyDeps wiring failure, does the record say where each exit is traced now that stdout goes to dobby.out and not dobby.log, and does an Acceptance criterion cover it."
  - "Decision 10, Decision 11 and the Acceptance criteria of decision-record.md: when lym adopt (internal/adoptcli/adopt.go) records an owner twice in a row, so two adopt checks can start while the template gate allows two slots, does the record say what keeps the check.json record and the suite.log file consistent, and what lym beam status shows between a spawn and the child's first write."
  - "Decision 9, Decision 10 and the Acceptance criteria of decision-record.md: lym adopt reconciles only through fabricengine.ReconcileOrigin and gatecli's loadGate reconciles nothing, so does the record say the adopt gate key is migrated from the hub's tuned finalize tiers before the adopt check reads it, on a hub whose build stamp is stale."
---

Each entry names a question the record must answer; none pre-states what the answer is.
