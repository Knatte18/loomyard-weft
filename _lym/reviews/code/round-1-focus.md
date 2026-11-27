---
round: 1
focus:
  - "internal/burlerengine auditFixRound and its SpawnedBySubagent skip: which subagents does the flag exempt (spawned by a fresh subagent, by a self-check fork, deeper chains), and does the read-only rule, the nested-spawn check and the fork count still hold for each case the amended board entry cares about?"
  - "The board entry's second item (may a fixer's general-purpose subagent spawn forks): read the decision the diff and change description take against what the entry asks for in either outcome, and check the stated reason, including the claim about what spawn guards deny today, against the code."
  - "Every consumer of AuditForks, AuditForksIncremental and ForkReport across the repository: how does each one's counting, violation checks or warnings change now that inherited tool calls are left out and SpawnedBySubagent exists, and does any need to read the new flag?"
  - "internal/shuttleengine/claudeengine/audit.go inherited-history exclusion: is the spawning transcript chosen correctly for every subagent shape (session-spawned, subagent-spawned, fork of a fork, missing or partial .meta.json), and does anything besides tool_use counting (write events, tool results, report-returned) still read inherited lines as the subagent's own?"
  - "The new error path for a missing or unreadable spawning transcript, together with the incremental audit's seen-transcript filter and transcripts still being written: when can it fail an audit, and how do callers handle that failure?"
  - "Tests: does the changed fixture setup in audit_test.go keep each existing assertion checking what its name states, and is every new behaviour covered, including exclusion against the session's own transcript for a session-spawned fork and the parentAgentId layout matching what real transcripts record?"
  - "Does each claim in _lym/landing/summary.md, including the Merge danger door and blast radius, match what the diff can affect?"
---

The diff is small and centred on the fork audit's attribution of tool calls, and on which subagents the fix-round audit exempts.
