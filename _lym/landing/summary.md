# Burler fix round: stop failing on a fresh subagent's writes

The burler twill fixer may start a fresh subagent to make its fixes, and that subagent writes on the fixer's behalf.
The fork audit counted every subagent transcript as a self-check fork.
It therefore read the fresh subagent's `Write`/`Edit` calls as a fork mutating a file.
The fix round then halted as `producer-hard-failure`, even though the fixer had done what it was asked.

The audit now tells a fork from a fresh subagent, and the fix round holds only the forks to the read-only rule.

Closes #144

## What changes

- `shuttleengine`: `ForkReport` gains `FreshSubagent`, and `ForkAudit` gains `NamedForkSpawns`.
  The `ForkAuditor` docs now say the audit covers every subagent, forks and fresh ones alike.
- `shuttleengine/claudeengine`: the audit reads the `.meta.json` beside each subagent transcript.
  It marks a transcript as fresh only on positive evidence:
  a parsed file that names a subagent type other than the fork type and does not flag a fork.
  A transcript with a missing, unparseable or kind-less `.meta.json` still reads as a fork, so the check stays strict.
  `NamedForkSpawns` counts the named spawns whose `subagent_type` is the fork type, via the existing `forkSubagentType`.
- `burlerengine`: `auditFixRound` drops fresh subagents before it checks and counts forks.
  A name given to a fresh subagent is no named-fork violation.
  Forks that write, nest `Agent` calls or carry names still fail the round, with the same way-forward text.
  The fork-count and unreturned-report warnings now cover the self-check forks only.

## Design decisions

- The fix is in the audit, not in the fixer's brief or in the guard's reaction.
  A fresh subagent is the fixer's way to write, so the audit separates the kinds of subagent
  instead of telling the fixer not to use one.
- The guard still fails the run on a real fork write.
  No revert or retry was added for that case.

## To check by hand

- `fix-round-named-fresh-delegate-fails-as-named-fork` (design, medium), in `auditFixRound`:
  a fixer that names a fresh delegate may still fail as a named fork.
- `fresh-subagent-fork-literal-not-constant` (consistency, blocking), in the audit tests and `claudeengine`:
  a fork-type literal where the `forkSubagentType` constant belongs.
- `fresh-subagent-test-bypasses-public-audit-surface` (design, medium), in the `claudeengine` audit tests:
  a fresh-subagent test that bypasses the public audit surface.
- `meta-without-kind-reads-fresh` (design, medium), in `auditForkTranscript`:
  a `.meta.json` without a kind may read as fresh instead of as a fork.
- Not run against a live Claude session: the new `.meta.json` fields (`agentType`, `isFork`) are read from Claude Code's
  on-disk layout, so check them against a real fixer run that starts a fresh subagent.
