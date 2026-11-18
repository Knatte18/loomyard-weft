# Burler fix round: audit only the twill fixer's self-check forks

Issue #144 reported a loom run that halted at Code-Burler-Fix as producer-hard-failure, because the fix-round audit said a twill fixer fork attempted 5 write/edit calls.
The fixer's session transcripts show that none of its four self-check forks wrote anything.
The writes came from two `general-purpose` subagents that the fixer started to make its fixes, which it is allowed to do.
The Claude fork audit reported every transcript under the session's `subagents/` directory as a fork,
so `auditFixRound` took the delegates' writes for a fork's writes and failed the half.

## What changes

- `shuttleengine.ForkReport` gains `FreshSubagent`, which is true for a subagent started with a fresh context and false for a fork that inherits the parent's context.
- `claudeengine` reads it from the `.meta.json` Claude Code writes beside each subagent transcript (`isFork`, `agentType`).
  A transcript with no readable meta file reads as a fork, so the audit stays strict when Claude Code records nothing.
  The meta read that already supplied the spawn-boundary `toolUseId` now returns both facts.
- `burlerengine.auditFixRound` checks and counts only the forks.
  A fresh subagent's writes are the fixer's own fixes, and the round's gate and later review rounds check them like any other fixer edit.
  A self-check fork that writes, nests a spawn or runs a mutating git command is still a hard error with the same way forward.

## Design decisions

- **Prompts unchanged.** The issue offered a stricter fork brief as one fix.
  The `burler-fix-self-check` boilerplate already says the forks are read-only and return their findings only in their final message, and the forks in the halted run obeyed it.
  The fault was in the audit, so the brief is left as it is.
- **No auto-revert or retry.** The issue also suggested having the guard revert or retry.
  With delegate writes no longer misread, a fork-write error now means a real fork wrote, and the manual revert-and-resume way forward still fits that case.
- **Review rounds and webster unchanged.** `auditClusterRound` still polices every subagent of a reviewer, since nothing in a review session may write.
  Webster's audits ignore the new field, so their behavior is the same.

## Tests

- `internal/burlerengine/cluster_test.go`: a fix-round audit that holds a fresh subagent with write calls beside two clean forks is clean, and it counts two forks.
- `internal/shuttleengine/claudeengine/audit_spawnboundary_test.go`: `FreshSubagent` is read from fork, `general-purpose`, fork-type-without-flag, missing and unparseable meta files.
