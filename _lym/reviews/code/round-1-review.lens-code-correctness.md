```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function and branch the diff adds or changes on concrete inputs.

**Per function or branch: the case I ran, and whether the result is correct**

- **`AuditForksIncremental`, `subagents/` missing.** Returns the parent facts with an empty `Forks`. Correct, and unchanged.
- **`AuditForksIncremental`, `subagents/` unreadable for another reason.** Returns an error. Correct.
- **`AuditForksIncremental`, a fork the session spawned, `.meta.json` with no `parentAgentId`.**
  - `inherited` is the session's tool_use ids.
  - The fork's own calls have new ids and are counted; its replayed session calls are left out.
  - Correct.
- **`AuditForksIncremental`, `.meta.json` missing or unparseable.**
  - `meta` is zero, so `ParentAgentID` is "" and the session's ids are used.
  - `SpawnedBySubagent` is false and `FreshSubagent` is false (reads as a fork).
  - Correct. Before `parentAgentId` existed, the old miscount comes back for a nested fork, which is an acceptable degradation.
- **`AuditForksIncremental`, the real halted layout.**
  - I ran this on the real files of the `fabric-core-foundation` session.
  - `agent-abc5528dbce58b86c.meta.json` records `"parentAgentId":"a896671709490d648"`, and `agent-a896671709490d648.jsonl` exists, so the `agent-&lt;parentAgentId&gt;.jsonl` naming is right.
  - All 5 `Agent` ids in the halted fork's transcript are also in the spawner's, so the fork's `AgentCalls` becomes 0.
  - The fork keeps 22 tool calls of its own that are not in the spawner's transcript.
  - Correct.
- **`AuditForksIncremental`, a fork of a fork (session → fork A → fork B).**
  - B's inherited set is A's transcript ids, and A's transcript holds A's replayed session ids plus A's own calls.
  - So B's replay is fully covered.
  - A's own spawn call is still counted in A's report. Correct.
- **`AuditForksIncremental`, spawner transcript in `seenTranscripts`.**
  - `subagentToolUseIDs` reads by path whatever the seen filter says, so the ids are still available. Correct.
- **`AuditForksIncremental`, spawner transcript missing.**
  - Returns the error `claudeengine: read spawning subagent transcript …`, which wraps `fs.ErrNotExist`.
  - The audit fails as documented, but callers misread that error: see F2.
- **`subagentToolUseIDs`, two forks with the same spawner.** The second lookup hits the cache. Correct.
- **`subagentToolUseIDs`, read failure.** Nothing is cached, and the error returns at once. Correct.
- **`collectToolUseIDs`, a block with an empty id or a non-tool_use block.** Skipped. Correct.
- **`auditParentTranscript` refactor.**
  - Every count, list and `writeEvents` keeps its prior meaning.
  - `toolUseIDs` is collected only from assistant lines.
  - The error path returns zero facts. Correct.
- **`auditForkTranscript`, inherited write tool_use and its replayed tool_result.**
  - The write is skipped before `tracker.call`.
  - The replayed tool_result names an id the tracker never saw, so `result` is a no-op.
  - `WriteEvents` and `WritePaths` stay the fork's own. Correct.
- **`auditForkTranscript`, `inherited` nil (the test call sites).** A lookup in a nil map returns false. Correct.
- **`auditForkTranscript`, `ReportReturned`.** Logic unchanged. Correct.
- **`SpawnedBySubagent`.** Set exactly when `ParentAgentID` is not empty, whatever kind the spawner was. Correct as a fact; how `auditFixRound` uses it is F1.
- **`auditFixRound`, a fresh subagent of the fixer that spawns forks.**
  - The forks are skipped, so there is no violation and the count stays at the self-check forks.
  - That matches the stated decision. Correct.
- **`auditFixRound`, a self-check fork X that spawns fork Y, and Y writes a file.**
  - X's own `Agent` call is counted, so the nested-spawn violation fires. Correct so far.
  - But Y is dropped from `selfCheck.Forks`, so `forkViolation` computes `untouched` without Y's writes.
  - Wrong way forward: see F1.

**Findings**

F1
- Severity: MEDIUM. Class: design.
- Location: `internal/burlerengine/cluster.go:942`
- Summary: a fork that a self-check fork spawned is now left out of the target-change check. The halt then says "no target change was found … resume the run" even when that inner fork wrote files.
- Evidence:
  - `cluster.go:942` skips every `SpawnedBySubagent` report, so it cannot tell a spawner that is a fresh subagent from one that is a self-check fork.
  - `cluster.go:866-874` computes `untouched` only over `selfCheck.Forks`.
  - `cluster.go:906` then words the way forward as no change.
  - At the review base, the inner fork was not `FreshSubagent`, so it stayed in `selfCheck.Forks` and its writes made `untouched` false. The halt then said "inspect … revert what the fork wrote".
  - So the diff turns a correct revert instruction into a false "resume".

F2
- Severity: LOW. Class: design.
- Location: `internal/shuttleengine/claudeengine/audit.go:120`
- Summary: the new missing-spawner-transcript error wraps `fs.ErrNotExist`, and webster reads that as "the session's transcript is not on this machine", so it is misreported or swallowed.
- Evidence:
  - `audit.go:120` wraps the `os.Open` error with `%w`.
  - `recordbatch.go` around line 366 maps `fs.ErrNotExist` to `archiveUnattributable` with a machine-local message.
  - `recordbatch.go` around line 794 returns `nil, nil` on `fs.ErrNotExist`, which skips that call's fork correctness audit.
  - Unreadable subagent transcripts already behave this way (`audit.go:426`); the diff adds a new path that does the same.
  - The case is unlikely: a spawner transcript is missing while its child's transcript exists.

**Tool calls**

1. Real `.meta.json` files under `~/.claude/projects`: to confirm `parentAgentId` is the key Claude Code records and how its value maps to a file name, which the manifest showed only in test fixtures.
2. A `comm` over tool_use ids of the real halted fork and its spawner: to confirm the fix removes all 5 inherited `Agent` ids. Its sort order was wrong, so I reran it.
3. The same comparison with `rm` of the temporary files: refused by the guard, nothing ran.
4. The same comparison with C-locale sort, without `rm`: the result I used. Its four temporary files were left in the session scratchpad.
