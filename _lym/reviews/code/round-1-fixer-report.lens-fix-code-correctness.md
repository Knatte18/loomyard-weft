```yaml
lens: fix-code-correctness
outcome: reported
```

Findings:

1. **LOW** — the F5 row does not pin `!meta.IsFork`, because the F2 change made it redundant for that row.
   - Changed text: the `ForkFlagWithoutAgentType` row in `TestAuditForks_FreshSubagentReadFromMeta` (`audit_test.go`), and the expression `metaFound && !meta.IsFork && meta.AgentType != "" && meta.AgentType != forkSubagentType` in `audit.go`.
   - The row's meta is `{"isFork":true,"toolUseId":...}` and has no `agentType`.
   - With `!meta.IsFork` deleted, `AgentType != ""` is still false, so `FreshSubagent` is still false and the row passes.
   - No row has `isFork:true` together with a non-fork `agentType`, so deleting `!meta.IsFork` still fails no test. That is the gap F5 named.
   - Fix: add a row such as `{"isFork":true,"agentType":"general-purpose"}` wanting false.

2. **LOW** — a named Agent call with no `subagent_type` is now counted as a named spawn but not as a named fork.
   - Changed text: `auditParentTranscript` (`audit.go`), where `namedForkSpawns++` fires only when `subagent_type == forkSubagentType`.
   - `auditFixRound` now sets `selfCheck.NamedSpawns = audit.NamedForkSpawns`, so such a call no longer fails the fix round.
   - This is consistent with the reviewer's F3 proposal and with `forkSubagentType`'s documented meaning. It matters only if Claude Code treats an omitted `subagent_type` as a fork.
   - No test pins either reading.

No other issues found:
- **FreshSubagent expression:** I ran every row through it and each returns the wanted value. That covers `{}`, `null` (`json.Unmarshal` of `null` succeeds and leaves empty fields), the toolUseId-only meta, the missing meta and `{`.
- **`auditParentTranscript` signature:** its only caller is `AuditForksIncremental`, which was updated at both return sites.
- **Fix-round tests:** the fresh-delegate row fails if `NamedSpawns` is still copied, and the named-fork row's error text matches `"burler: twill fixer 1 fork(s) were spawned with a name"`.
- **`TestAuditForks_NamedForkSpawnsCountOnlyNamedForks`:** it expects 3/2/1 and would fail if every named call counted as a fork.
- **Other `ForkAuditor` implementers:** the only other one is `shuttlefake`, a test fake, and no test relies on it setting `NamedForkSpawns`.
- **Webster:** it reads `NamedSpawns` and is unaffected.

Tool calls:
1. `grep -rn "func (.*) AuditForks(" --include=*.go internal`, to find every `ForkAuditor` implementer, which my context lacked.
