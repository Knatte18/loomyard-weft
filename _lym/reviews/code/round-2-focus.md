---
round: 2
focus:
  - "internal/shuttleengine/claudeengine/audit.go, auditForkTranscript's FreshSubagent derivation: does it name the fork subagent type only through the package constant forkSubagentType (background.go), and does a subagent meta recording neither isFork nor agentType read as a fork, as ForkReport's doc in internal/shuttleengine/forkaudit.go says?"
  - "internal/burlerengine/cluster.go, auditFixRound together with auditParentTranscript in claudeengine/audit.go: does a fixer that spawns a named fresh delegate pass the fix-round audit, while a named fork spawn still fails it, and do auditClusterRound and webster's audit read NamedSpawns as before?"
  - "internal/shuttleengine/claudeengine tests for FreshSubagent classification: are they driven through AuditForks, does every discriminator term of the derivation (isFork, agentType, a missing meta, an unparseable meta) have a row that fails when the term is removed, and does each test the redundancy report lists as contained carry a //testtiming:keep reason naming what it pins?"
  - "internal/shuttleengine/engine.go ForkAuditor docs and the audit.go file header, AuditForks and AuditForksIncremental docs: do they describe every subagent of the session, forks and fresh alike, consistent with the ForkAudit.Forks doc in forkaudit.go?"
---

Focus entries are drawn from the open findings' sites, restated without reference to the review.
