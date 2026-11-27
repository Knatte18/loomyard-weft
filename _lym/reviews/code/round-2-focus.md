---
round: 2
focus:
  - "internal/burlerengine/cluster.go, auditFixRound: which fork reports are left out of selfCheck.Forks, and do the file writes and git-mutating commands of every fork descending from a self-check fork, at any depth or with an unresolved spawner, still reach targetUntouched and the way forward fixViolationWayForward gives? Does a fix fork under a fixer's fresh subagent stay allowed? Check that cluster_test.go covers these cases."
  - "internal/shuttleengine/claudeengine/audit.go and audit_nested_test.go: how is the spawner of a fork identified, which inherited tool_use ids are left out for a session-spawned and for a subagent-spawned fork, and does a test fail if either exclusion is removed?"
  - "internal/shuttleengine/claudeengine/audit.go, subagentToolUseIDs: how do the callers of AuditForks and AuditForksIncremental (internal/websterengine/recordbatch.go and runwrites.go, internal/shuttleengine wait.go) treat a missing or unreadable spawner transcript?"
  - "Do these texts match what the code does: contracts/stencils/burler/burler-fix-self-check.md on who may spawn, the doc comments in internal/shuttleengine/forkaudit.go and internal/burlerengine/doc.go on what the fork reports hold and who may spawn, and the Merge danger section and other claims in _lym/landing/summary.md?"
---
