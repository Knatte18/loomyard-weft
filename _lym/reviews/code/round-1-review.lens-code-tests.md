```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it**

1. `readSubagentMeta` still supplies the spawn-boundary `toolUseId` (refactor of `forkSpawnToolUseID`): `TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded` (`AgentCalls == 0`) and `TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything` (`AgentCalls == 1`).
2. `FreshSubagent = true` for a `general-purpose` meta: `TestAuditForkTranscript_FreshSubagentReadFromMeta` row `GeneralPurposeSubagent` (`audit_spawnboundary_test.go:89`).
3. Meta with `agentType: "fork"` reads as fork, with or without `isFork`: rows `Fork`, `ForkTypeWithoutForkFlag`.
4. Missing or unparseable meta reads as fork: rows `NoMetaReadsAsFork`, `UnparseableMetaReadsAsFork`.
5. The `!meta.IsFork` term on its own (a meta with `isFork: true` and an `agentType` other than `"fork"` reads as fork): **none**. Every fork row also carries `agentType: "fork"`, so deleting `!meta.IsFork` from `audit.go:328` fails no row.
6. `auditFixRound` does not treat a fresh subagent's writes as a violation: `TestAuditClusterRound` row "fix: a fresh subagent that wrote is neither a violation nor a fork" (`cluster_test.go:99-107`, `WriteCalls: 5`, no `wantErr`).
7. `auditFixRound` excludes fresh subagents from the fork count: same row, `wantForks: 2` with three reports, and `wantN: 2` with no count warning expected (`cluster_test.go:219`, `:248`).
8. `auditFixRound` excludes fresh subagents from unreturned-report warnings: same row — the fresh report's `ReportReturned` is zero-value `false` and `wantWarnings` is empty (`cluster_test.go:248-250`).
9. A self-check fork's write still fails the fix round: existing row with `errSubstr` ``twill fixer fork "b" attempted 2 write/edit tool call(s)`` (`cluster_test.go:~95`). No fresh subagent in that row, but an inverted or dropped filter would break it or the fresh row.
10. The review-round audit (`auditClusterRound`) still counts and polices every subagent, fresh ones included — claimed unchanged in the commit message and summary: **none**. No non-fix row carries `FreshSubagent: true`, so applying the filter in `auditClusterRound` too fails no test.
11. `FreshSubagent` through the public `AuditForks`/`AuditForksIncremental` surface from a real `subagents/` directory: **none directly**; pinned only through the unexported `auditForkTranscript`, which those methods call without branching (`audit.go:83`), so the unit row suffices.

**Redundancy-report candidates**

- `TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded` (contained, covered by `TestAuditForkTranscript_FreshSubagentReadFromMeta`): the covering test asserts only `FreshSubagent`; the candidate asserts `AgentCalls == 0`, `ToolCalls["Agent"] == 0`, `ToolCalls["Read"] == 1`, `len(BashCommands) == 1`, and `ReportReturned` — none of which the covering test checks.
- `TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything` (contained, same cover): asserts `AgentCalls == 1` (no meta → nothing excluded); the covering test's `NoMetaReadsAsFork` row checks only `FreshSubagent`.

**Findings**

- **LOW · design** · `internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:85-99` — No row pins the `!meta.IsFork` term of the `FreshSubagent` derivation; add a row with `isFork: true` and a non-`fork` `agentType` wanting `false`. Evidence: `audit.go:328`, table rows at `audit_spawnboundary_test.go:89-93`.
- **LOW · design** · `internal/burlerengine/cluster_test.go:99-107` — No review-round row shows `auditClusterRound` still fails on and counts a fresh subagent's write, the behavior the commit message claims unchanged; add a non-`fix` row with a `FreshSubagent: true` report carrying `WriteCalls`, wanting the write error. Evidence: `cluster.go:488-500`, commit `427e13d4` message.
- **NIT · consistency** · `internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:37` — Redundancy report lists `TestAuditForkTranscript_SpawnBoundaryAgentCallExcluded` as contained, but it pins what its cover does not (the spawn-boundary Agent call's exclusion, tool counts, `ReportReturned`); mark it `//testtiming:keep pins the spawn-boundary Agent call's exclusion and the fork's own tool counts`. Evidence: `audit_spawnboundary_test.go:46-64` vs `:158`.
- **NIT · consistency** · `internal/shuttleengine/claudeengine/audit_spawnboundary_test.go:68` — Redundancy report lists `TestAuditForkTranscript_MissingMetaFallsBackToCountingEverything` as contained, but it pins `AgentCalls == 1` without a meta file, which its cover does not check; mark it `//testtiming:keep pins that no meta file excludes no Agent call`. Evidence: `audit_spawnboundary_test.go:79-81` vs `:158`.

Tool calls:
- `sed -n 205,262p internal/burlerengine/cluster_test.go` — the table loop's assertions on forks/warnings/errors; my context held only the loop's start, not what it asserts.
