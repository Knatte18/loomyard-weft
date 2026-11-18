---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/audit.go `FreshSubagent` derivation: settle whether `metaFound && !meta.IsFork && meta.AgentType != \"fork\"` classifies every real `.meta.json` shape correctly, including a fork whose meta omits `isFork` or `agentType`, so a real self-check fork is never read as fresh and skipped by the write audit."
  - "internal/burlerengine/cluster.go `auditFixRound`: settle whether excluding fresh subagents from `forkViolation` leaves the read-only contract enforced for self-check forks, and whether the fork count and unreturned-fork warnings still mean what the callers and warnings expect."
  - "Scope against the answer key: the board entry offers a prompt contract, a writing session, a guard that blocks or reverts, or a one-time retry; settle whether the diff's chosen route (exempting fresh subagents from the audit) is licensed by the entry, and whether it leaves the fix-round fork's own prompt unchanged without losing the no-mutation contract."
  - "Other consumers of `ForkAudit.Forks` (the review-round audit, webster's audits, any other shuttleengine caller): settle whether the widened meaning of `Forks` (every subagent, not only forks) changes their behavior, as the commit message claims it does not."
  - "Tests in audit_spawnboundary_test.go and cluster_test.go: settle whether the new behavior (fresh subagent excluded, missing meta read as fork, self-check fork write still failing the round) is covered, and check the redundancy report's contained candidates against their covering tests' assertions."
---

Seed focus set from the committed diff, the commit message and the board entry in the decision record.
