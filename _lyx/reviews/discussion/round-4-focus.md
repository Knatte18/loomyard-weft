---
round: 4
focus:
  - "In decision-record.md, the Decision on the fixer fork and interim notices (D5) and its matching Open risk: when a fixer's interim notice reaches Merriam on a gated run, does the stated bound match what the verify gate in internal/websterengine/verifygate.go does on a dirty tree (immediate failure, verify-gate report rewritten, finalize removing the strand when the attempt budget is spent)? Settle whether the Decision states the fixer being torn down mid-edit and either accepts it with a reason or names a remedy."
  - "In decision-record.md, the Decision on reading a held foreground tool call in a fork (D9) and its acceptance criterion: does the detection rule hold when the fork's transcript has a tool_use with no tool_result but a later entry (a sibling call's result or an attachment line) is newer than it? Settle whether the rule text and the acceptance criterion describe the same condition."
  - "In decision-record.md, the Decision that makes claudeengine return a read-time value or the newest fork-transcript write as the transcript time: does the Scope list every documented provider-neutral contract in internal/shuttleengine (the TurnEndActivity.TranscriptModTime field doc, the AgentActivity.LastActivity doc, doc.go) that this changes the meaning of?"
---

Entries target the three keys that were open in round 3; none caps severity.
