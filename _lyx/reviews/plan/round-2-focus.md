---
round: 2
focus:
  - "Where the tokencount golden `reads.json` lives and which test packages (`internal/shuttleengine/claudeengine`, `tools/tokencount`) read it: settle whether any fixture used by two or more packages still sits outside `internal/testkit/<kit>/`, per `PATTERN-testkit`."
  - "The id-less message key fallback in the Claude transcript reader card and the cross-transcript dedupe in the tokencount card (engine name plus message key): settle whether two distinct messages from different transcripts of one run can still get the same key."
  - "The `TestSessionUsage_SumsParentAndForks` rows in the Claude transcript reader card against `SessionUsage`'s current handling of entries under `subagents/` in `internal/shuttleengine/claudeengine/usage.go`: settle whether every row matches behavior the decision record keeps."
---

Focus targets the areas touched by the round 1 fixes.
