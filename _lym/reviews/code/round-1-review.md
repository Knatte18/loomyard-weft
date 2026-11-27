---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/cluster.go#auditFixRound
    summary: A fork that a self-check fork spawned is now dropped from the fix-round audit, so the target-change check ignores its writes and the halt falsely says no target change was found and tells the operator to resume.
    fix: "Make the exemption depend on the spawner's kind. In internal/shuttleengine/forkaudit.go, give ForkReport the spawner's transcript path, for example SpawnerTranscriptPath, empty when the session spawned it. In internal/shuttleengine/claudeengine/audit.go, fill it from meta.ParentAgentID as subagentsDir/agent-<ParentAgentID>.jsonl. In auditFixRound, exempt a SpawnedBySubagent report only when its spawner chain, followed through the audit's reports by that path, reaches a FreshSubagent report. Keep every other SpawnedBySubagent report, such as a fork spawned by a self-check fork, in selfCheck.Forks, so its writes and git commands are checked and count toward targetUntouched. Add a cluster_test.go row: a self-check fork with AgentCalls 1, plus a fork it spawned that wrote a file. The row expects the nested-spawn error whose way forward says to revert, not that no target change was found."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/audit_nested_test.go#TestAuditForks_SubagentForkCountsOnlyItsOwnCalls
    summary: No test pins that a fork the session spawned leaves out session tool calls it replays beyond its spawning call; with inherited set to nil for such forks, every test still passes.
    fix: "Add a row to audit_nested_test.go: a session-spawned fork with no parentAgentId whose transcript replays a session Bash call ('git commit -am x') and an Agent call whose ids the session transcript holds and the fork's toolUseId does not name, then makes one Read of its own. Assert that the fork's report has AgentCalls 0, no BashCommands, ToolCalls[\"Read\"] 1 and SpawnedBySubagent false."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/audit.go#subagentToolUseIDs
    summary: The new missing-spawner-transcript error wraps fs.ErrNotExist. Webster's record-batch therefore reads it as the session's transcript being absent from this machine, archiving the batch as unattributable on one path and silently skipping the fork audit on the other.
    fix: "In subagentToolUseIDs, wrap the read error so errors.Is(err, fs.ErrNotExist) is false for it. For example, format the underlying error with %v instead of %w, and add a comment saying a missing spawner transcript is an incomplete audit, not a session transcript absent from this machine. Extend TestAuditForks_MissingSpawningSubagentTranscriptErrors to assert that errors.Is(err, fs.ErrNotExist) is false."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-fix-self-check.md:48
    summary: "The stencil's sentence that the fixer is the only session that spawns contradicts the diff's decision that a fixer's fresh subagent may spawn forks of its own."
    fix: "Replace line 48 with: \"A fork spawns nothing; only the fixer and the subagents it starts to make its fixes spawn.\""
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:44
    summary: "The Merge danger blast radius names only burler. The diff also changes claudeengine's fork audit, which webster's record-batch and recover paths and shuttle's fork-authorized run finish read: their counts, write lists and Bash lists change, and they gain a new failure for a missing spawning transcript."
    fix: "Change the blast radius line to name burler's fix-round audit, webster's record-batch fork audit and shuttle's fork-authorized run finish, all through claudeengine's fork audit."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/forkaudit.go:26
    summary: "The ForkAudit doc says Forks holds one report per subagent the parent spawned. Forks also holds subagents other subagents spawned, which the new SpawnedBySubagent field marks."
    fix: "Change line 26 to: \"Forks holds one report per subagent of the session, whether the session or another subagent spawned it, forks and fresh subagents alike.\""
---

### [MEDIUM] A fork nested under a self-check fork escapes the target-change check

**Location:** `internal/burlerengine/cluster.go#auditFixRound`, line 942, where `SpawnedBySubagent` reports are skipped.

**Issue:** `auditFixRound` drops every `SpawnedBySubagent` report from `selfCheck.Forks`, whoever the spawner was.
The board entry's second item is settled as "a fixer's fresh subagent may spawn forks",
but the flag also covers a fork that a self-check fork spawned.
The self-check fork's own Agent call still raises the nested-spawn violation.
The problem is that `forkViolation` computes `targetUntouched` only over `selfCheck.Forks` (`cluster.go:864-871`).
A nested fork's file writes and git-mutating commands therefore no longer reach that flag,
and `fixViolationWayForward` (`cluster.go:906`) tells the operator "no target change was found … resume the run".
At the review base, that nested fork was not `FreshSubagent` and stayed in the set, so the halt correctly said to revert what the fork wrote.
The diff turns that correct instruction into a false one.
The case is rare, since it needs a self-check fork that already breaks the rule by spawning, but the way forward exists to be right in exactly such a breach.

### [LOW] The session-transcript exclusion for a session-spawned fork has no test

**Location:** `internal/shuttleengine/claudeengine/audit_nested_test.go#TestAuditForks_SubagentForkCountsOnlyItsOwnCalls`

**Issue:** `AuditForksIncremental` passes the session's tool_use ids as `inherited` for every fork without a `parentAgentId` (`audit.go:90`).
No fixture replays a session call other than the spawning one, which `toolUseId` already excludes.
The write-events fixture was rewritten to avoid the collision rather than assert it (`audit_test.go:453-466`).
Replacing `parent.toolUseIDs` with nil passes every test.
On the real halted session, a depth-1 fork shares only its spawning call with the session transcript.
That makes this a low-impact gap: what the exclusion adds today is mainly that a fork with no `.meta.json` also has its spawning call left out.

### [LOW] A missing spawning transcript is misread by webster as a missing session transcript

**Location:** `internal/shuttleengine/claudeengine/audit.go#subagentToolUseIDs`, line 120.

**Issue:** The error wraps `os.Open`'s error with `%w`, so `errors.Is(err, fs.ErrNotExist)` holds.
`websterengine/recordbatch.go` maps that to `archiveUnattributable`, whose message claims the session's transcript is absent from this machine (around line 366).
Its recover-side caller returns `nil, nil` on the same error (around line 794), which skips that call's fork correctness audit.
The path is new in this diff.
It is unlikely to occur, since a child transcript implies its spawner's.

### [LOW] The fixer self-check stencil still says only the fixer spawns

**Location:** `contracts/stencils/burler/burler-fix-self-check.md:48`

**Issue:** "A fork spawns nothing, so the fixer is the only session that spawns" no longer holds now that the diff, through `doc.go` and `auditFixRound`, lets a fixer's fresh subagent spawn forks.

### [LOW] The Merge danger blast radius names less than the diff reaches

**Location:** `_lym/landing/summary.md:44`

**Issue:** The door call `two-way` holds: `ForkAudit` is in-memory, no record format or config meaning changes, and nothing is pushed.
The blast radius `burler` leaves out the other readers of `AuditForks` and `AuditForksIncremental`.
These are webster's `recordbatch.go` and `runwrites.go`, and shuttle's `wait.go:2082`.
Their fork counts, write lists and Bash lists now leave out inherited calls, and each gains the new spawning-transcript error path.

### [NIT] The ForkAudit doc understates what Forks holds

**Location:** `internal/shuttleengine/forkaudit.go:26`

**Issue:** "Forks holds one report per subagent the parent spawned" is contradicted by the new `SpawnedBySubagent` field on the reports it holds.

## Ground truths checked

- I checked the real `.meta.json` files of the halted `fabric-core-foundation` session.
  They record `parentAgentId` as the spawner's agent id, and the spawner's transcript is `agent-<id>.jsonl`, the layout the fixture and `subagentToolUseIDs` assume.
- In the halted fork `agent-abc5528dbce58b86c`, every one of the 5 Agent tool_use ids, 9 Bash and 11 Read ids also appears in its spawner's transcript.
  Its own calls, 18 Bash and 4 Read, appear only in the fork's transcript.
  So the fix brings its AgentCalls to 0 and keeps the spawner's Bash commands out of its git-mutating check.
- Focus item 2: the diff takes the "may" outcome, which the board entry allows, so no refusal or guard is owed.
  The claim that nothing denies the Agent tool today matches `claudeengine/config.go:57-61`, where the retired `deny_agent_tool` key "switches nothing".
- The summary's figure "the later fork reports 2 Agent calls and 1 Bash command" on the old code is right for the fixture: `toolu_FirstFork` plus `toolu_LastOwnSpawn`, with `toolu_LastFork` excluded by `toolUseId`.
- The `Root-Cause:` trailer on `fa94897ed` names the defect the diff removes.
  No test assertion was loosened: the spawn-boundary and streaming tests changed only their call sites.
  The write-events fixture keeps every assertion and gives the fork distinct ids.

## Rejected

- lens:code-tests LOW, that no recording backs the `parentAgentId` key or the `agent-<id>.jsonl` layout: rejected, because the real session metas named above record exactly that key and layout.
- lens:code-tests MEDIUM severity for the session-spawned exclusion gap: kept as F2 but lowered to LOW, because real depth-1 forks replay only their spawning call, which `toolUseId` already covers.
- lens:code-tests side remark that the old code gave `agent-last` 3 Agent calls: not a finding, and wrong, since the old code excluded `toolu_LastFork` by `toolUseId` and counted 2.
