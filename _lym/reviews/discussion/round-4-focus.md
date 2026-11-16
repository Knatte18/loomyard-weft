---
round: 4
focus:
  - "D12, the Codex batching nudge: the Codex rollout holds top-level function_call items (spawn_agent and send_message in the collaboration namespace) beside custom_tool_call items named exec. Settle whether the D12 rules for grouping a payload into a cell, finding its call in the rollout, and ending a lookup or edit chain are stated for a payload that is not a nested exec call, and whether any sentence still says every model call or every tool_use_id is an exec cell."
  - "D1 and D4, the producer table and prompt-engine scan: internal/shedadapters/bouncer.go builds shuttleengine.Spec literals that reach only an attach probe (awaitLiveJudge, awaitLiveSeed) and others that fill the prompt inline in the launching function (judgeCall, runSeedSpawn). Settle whether the scan's test for a launch spec and the table's renderer rule hold for both kinds."
  - "D12, the read-again rule: settle whether the Codex rule states how a sliced Bash read's file is compared with an earlier read's path, and whether it applies the same unchanged-since test as the Claude rule in internal/shuttleengine/claudeengine/batchingnudge.go."
---

Each entry names a place the round-3 review found wrong.
The rewritten text there is new and needs a fresh read.
