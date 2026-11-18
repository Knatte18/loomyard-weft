```yaml
lens: code-correctness
outcome: reported
```

**Cases run, per function or branch**

- `readSubagentMeta`, meta file missing: `os.ReadFile` errors → `({}, false)`. Correct.
- `readSubagentMeta`, meta `{`: `json.Unmarshal` errors → `({}, false)`. Correct.
- `readSubagentMeta`, meta `null`, `{}` or `{"toolUseId":"toolu_x"}`: Unmarshal succeeds, zero value → `(meta, true)`. Correct for the reader; see finding 1 for the caller.
- `auditForkTranscript` `FreshSubagent`, real fork meta (`agentType:"fork"`, `isFork:true`): false. Correct.
- `FreshSubagent`, `general-purpose` meta, no `isFork`: true. Correct.
- `FreshSubagent`, `agentType:"fork"` without `isFork`: false. Correct.
- `FreshSubagent`, `isFork:true` without `agentType`: `!true` → false. Correct.
- `FreshSubagent`, meta present but carrying neither `isFork` nor `agentType`: `true && !false && "" != "fork"` → true. **Wrong** (finding 1).
- `FreshSubagent`, no / unparseable meta: `metaFound` false → false. Correct.
- `auditForkTranscript` spawn-boundary id with missing meta: `meta.ToolUseID` "" → nothing excluded, as before. Correct.
- `auditFixRound`, nil audit: count warning, 0. Correct (unchanged).
- `auditFixRound`, `&ForkAudit{}`: `selfCheck.Forks` nil, no violation, forks 0, count warning. Correct.
- `auditFixRound`, two clean forks + one fresh with `WriteCalls:5`: filter keeps 2, no violation, forks 2, no warning. Correct.
- `auditFixRound`, only fresh subagents: forks 0 → "spawned 0 fork(s)" warning. Correct.
- `auditFixRound`, fresh subagent with `ReportReturned:false`: no unreturned warning (by design). Correct.
- `auditFixRound`, fork with `WriteCalls>0` beside a fresh one: error with `len(selfCheck.Forks)`. Correct.
- `auditFixRound`, shallow copy `selfCheck := *audit` then reassigning `Forks`: caller's audit not mutated. Correct.
- `auditFixRound`, `audit.NamedSpawns > 0` where the named spawn was a fresh subagent: `selfCheck` keeps the parent-wide `NamedSpawns`, `forkViolation` hard-fails the round. **Wrong** (finding 2).
- `auditFixRound`, self-check fork that itself spawns a general-purpose subagent: the fork's `AgentCalls>0` still trips `forkViolation` even though the nested transcript (read as fresh) is skipped. Correct.

**Findings**

1. MEDIUM · design · `internal/shuttleengine/claudeengine/audit.go:328` — A meta file that parses but records neither `isFork` nor `agentType` (`null`, `{}`, or `{"toolUseId":"…"}` — the only field the code read before this diff, old `forkSpawnToolUseID` diff 002 lines 17-49) reads as a fresh subagent, so the fix-round audit silently skips a self-check fork's writes. This fails open and contradicts the `ForkReport` contract at `internal/shuttleengine/forkaudit.go:37-39` ("A transcript whose kind the provider does not record reads as a fork"). All real fork metas on this machine carry both keys, so today's exposure is a Claude Code format change; the fail-safe derivation is to read fresh only when `agentType` is present and ≠ "fork" and `isFork` is not true.
2. MEDIUM · design · `internal/burlerengine/cluster.go:552-559` — The diff filters `Forks` but `selfCheck.NamedSpawns` still counts every named Agent call of the fixer, fresh-subagent spawns included (`claudeengine/audit.go` `auditParentTranscript`, `namedSpawns++` on any Agent block with a `name`). A fixer that passes `name` to a general-purpose delegate therefore still fails the run as producer-hard-failure with "named forks silently lose inherited context" — the same misattribution of delegate behavior to the self-check forks this diff exists to remove, and its way forward ("revert what the fork wrote") is wrong for that case.

No tool calls.
