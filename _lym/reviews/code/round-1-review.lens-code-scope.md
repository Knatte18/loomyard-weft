```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points.

**Decisions** (a decision record alone; the record has no `Added after Discussion` entries)
- **G1:** the nested-spawn check leaves out tool_use ids that the parent transcript also holds. Its fixture is a subagent that spawns several forks in turn.
- **G2:** decide whether a fixer's general-purpose subagent may spawn forks. If it may not, the refusal names that subagent and the spawn guard denies the call.

**Per change**
- **C1:** inherited-history exclusion in `claudeengine/audit.go`. This covers the `parentAgentId` meta field, `subagentToolUseIDs`, `collectToolUseIDs`, `auditForkTranscript`'s new `meta`/`metaFound`/`inherited` parameters and the spawner-id cache (audit.go:77-133, 340-341, 367-399). Traces to G1.
  - It leaves inherited ids out of every count, Bash and writes included, not only Agent calls. That is the same mechanism and the same reason G1 states ("inherited history, not the fork's attempts"), and an inherited Bash `git commit` would trip the git-mutating check the same way. It is in scope.
- **C2:** `auditParentTranscript` returns `parentTranscriptFacts` instead of eight values (audit.go:271-330). Forced by C1, which needs the parent's tool_use id set returned.
- **C3:** `AuditForksIncremental` builds the `ForkAudit` value once, ahead of the early return for a missing `subagents/` (audit.go:54-73). Forced by C2: the call site's eight-value destructure goes away.
- **C4:** `ForkReport.SpawnedBySubagent` (forkaudit.go:56-57), set at audit.go:378. Traces to G2, which takes the "may" outcome; it is also the signal C5 reads.
- **C5:** `auditFixRound` skips subagents with `SpawnedBySubagent` set, plus its doc comment (cluster.go:932-942) and `burlerengine/doc.go:111`. Traces to G2.
- **C6:** `audit_nested_test.go`, with its gp → first and last fork fixture and the missing-spawner error test. Traces to G1's fixture requirement and covers C1's new error path.
- **C7:** the `cluster_test.go` row for a fork a fresh subagent spawned. Traces to G2 and covers C5.
- **C8:** `auditForkTranscript` signature updates in `audit_spawnboundary_test.go` and `audit_streaming_test.go`. Forced by C1.
- **C9:** `audit_test.go`'s `TestAuditForks_WriteEventsCarryTimeAndResult` gives the fork transcript its own ids, `toolu_fork_w`, in commit 1ca04c717. Forced by C1: identical ids would now be excluded as inherited.
- **C10:** commit 10550ef0a re-wraps comments inside the files above. These are edits to traced changes, so they are forced or cosmetic.

**Per decision**
- G1: built by C1, C2, C3, C6, C8 and C9.
- G2: built by C4, C5 and C7. It is decided as "may", and the reason sits in `_lym/landing/summary.md` under Design decisions. Because the outcome is "may", the entry's conditional refusal and spawn guard are not required.

**Findings:** none from this lens. Every change traces to G1 or G2 or is forced by one, and both goal points are built.

One thing outside this lens, for the correctness lens or the handler: `auditFixRound` takes `SpawnedBySubagent` reports out of `selfCheck`. So `forkViolation`'s `targetUntouched` (cluster.go:864-873) no longer sees writes by a fork nested under a self-check fork. The way-forward text can then say "no target change was found" when such a fork wrote a file.

Tool calls: none.
