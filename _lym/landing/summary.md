# Count only a fork's own tool calls in the fork audit

A burler fix round could halt `failed`, blaming a fork for nested spawns it never attempted.
A fork spawned by a subagent replays that subagent's earlier tool calls in its transcript,
so the later of several forks carried its siblings' `Agent` calls.
The audit excluded only the fork's own spawning call.

Closes #276

## What changes

- `claudeengine`'s fork audit reads each subagent's spawning transcript:
  the one `parentAgentId` in its `.meta.json` names, or the session's own.
  Every tool call whose id that transcript also holds is left out of the report.
  This covers `Bash` commands, writes and other tool counts as well as `Agent` calls,
  for every reader of `AuditForks` and `AuditForksIncremental` in burler, webster and shuttle.
- A subagent whose spawning subagent's transcript is missing or unreadable now fails the audit.
- `ForkReport` gains `SpawnerTranscriptPath`, empty when the session spawned the subagent.
- A fix round's fresh (non-fork) subagent may spawn forks of its own.
  The fix-round audit neither checks nor counts them.
- The `burler-fix-self-check` stencil no longer says the fixer is the only session that spawns.

## Design decisions

- A fixer's fresh subagent is allowed to spawn forks, so no refusal or spawn guard is added for it.
- The fix-round exemption follows the spawner chain to the subagent the session itself spawned.
  Everything under a fresh subagent is exempt.
  Everything under a self-check fork is held to the read-only rule, a fresh subagent included,
  so a write or commit there still gets the way forward that says to revert it.
- The missing-spawner error does not match `fs.ErrNotExist`.
  Webster reads `fs.ErrNotExist` as the session's own transcript being absent from this machine.

## To check by hand

- A fork nested under a self-check fork must not escape the target-change check,
  in `auditFixRound` (`internal/burlerengine/cluster.go`).
  The change handles this with the spawner-chain walk there; confirm that it does.
  The nested fork's file writes and repository-changing git commands must count,
  so the halt does not say no target change was found.

## Merge danger

**Door:** two-way
**Blast radius:** `fork-audit`

The fork audit is `claudeengine`'s, and burler, webster and shuttle read it.
