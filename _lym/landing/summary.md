# Leave a fork's inherited history out of its fork audit

A twill fix round halted with "fork … attempted 4 nested spawn call(s)" against a fork that never spawned anything.
The fork had been spawned, fifth of five, by a general-purpose subagent of the fixer.
A fork's transcript replays its spawner's earlier tool calls with the fork's own agent id,
so the fifth fork's transcript held the spawn calls of its four older siblings.
The audit excluded only the fork's own spawning call, by the `toolUseId` in its `.meta.json`, and counted the other four as the fork's.

## What changes

- `claudeengine`'s fork audit reads `parentAgentId` from each subagent's `.meta.json`.
  It collects the tool_use ids of the spawning transcript: the session's own transcript, or `subagents/agent-<parentAgentId>.jsonl` for a subagent spawned by another subagent.
  Every tool call in the fork's transcript whose id that set holds is left out of every count, not only Agent calls:
  inherited Bash commands and writes were miscounted the same way.
  Each spawning subagent's transcript is read once per audit.
  A spawning subagent's transcript that is missing or unreadable fails the audit, as an unreadable subagent transcript already does.
- `shuttleengine.ForkReport` gains `SpawnedBySubagent`, true for a subagent another subagent spawned.
- `burlerengine`'s `auditFixRound` neither checks nor counts a subagent with `SpawnedBySubagent` set.

## Design decisions

- **A fixer's fresh subagent may spawn forks.**
  The fixer's fresh subagents already write on its behalf and go unaudited.
  Forks such a subagent spawns are part of its work, not the fixer's self-check forks,
  so they neither trip the read-only rule nor inflate the lens count.
  Forbidding them would need a guard that denies the Agent tool inside a subagent, and nothing denies that tool today.
  A self-check fork that spawns is still caught by its own spawn call, which is not inherited history.
- The spawning-call exclusion by `toolUseId` stays beside the new exclusion,
  so a fork whose spawning transcript lacks the call still has that call left out.
- `auditParentTranscript` now returns a struct of its facts in place of eight return values, since it also returns the id set.

## Tests

- `TestAuditForks_SubagentForkCountsOnlyItsOwnCalls` builds a general-purpose subagent that spawns two forks in turn.
  The later fork replays the spawner's Bash command and both spawn calls.
  Each fork reports only its own Read and, for the later one, its own Agent call; the spawner keeps its two spawn calls.
  On the code before this change, the later fork reports 2 Agent calls and 1 Bash command where it made 1 and 0.
- `TestAuditForks_MissingSpawningSubagentTranscriptErrors` pins the error for a missing spawning transcript.
- A `cluster_test.go` row pins that a fork spawned by a subagent, writes and a mutating git command included, is no violation and no self-check fork.

## Merge danger

**Door:** two-way
**Blast radius:** burler
