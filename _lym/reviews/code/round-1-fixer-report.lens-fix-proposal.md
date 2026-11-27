```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as proposed (F2–F6; F1 was fixed differently).

**Findings**

- **LOW**, `_lym/landing/summary.md`, "Blast radius" line (F5): the proposed list leaves out two readers of the changed audit, and narrows the old `burler` to the fix round only.
  - The cluster review round also reads it: `internal/burlerengine/handoff.go:44` calls `auditClusterRound(reviewer.ForkAudit, …)`, which runs `forkViolation` over the same `AgentCalls`, `WriteCalls` and `BashCommands`.
  - Webster's run-exit whole-session audit also reads it: `internal/websterengine/runlevel.go:1045-1116` runs `CheckParent` and loops over `result.ForkAudit.Forks`.
  - "shuttle's fork-authorized run finish" names where that audit is produced, not these two checks. "burler's fork audits" and "webster's record-batch and run-exit audits" would cover them.

F2, F3, F4 and F6: no findings.

- **F3:** under `%v`, `recordbatch.go:369` and `auditTerminalFork` (`recordbatch.go:794`) now return the error for a missing spawner transcript. A missing session transcript still matches `fs.ErrNotExist` through `auditParentTranscript`.

**Tool calls**

1. Searched for `AuditForks` and `SettleRetry` callers and read `SettleRetry` and the finish step in `wait.go`; my context showed only some of the callers.
2. Read `recordbatch.go` around line 792; I had not seen the second caller's function.
3. Searched for readers of `ForkAudit`, `auditClusterRound` and `auditFixRound`; my context did not show who reads the audit after the run finishes.
