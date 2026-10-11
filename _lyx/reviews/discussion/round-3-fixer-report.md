# Round 3 fixer report — discussion

## Changes

- F1 (MEDIUM, design), fixed as proposed.
  Premise verified: `newVerifyGate` in `internal/websterengine/verifygate.go` fails at once on `s.dirty()` and writes the verify-gate report on every failure; `handleGatedBoundary` finalizes `OutcomeDone` on a spent budget, and `finalize` calls `RemoveStrand` when `KeepPane` is false, which no webster spec sets.
  D5's fixer bullet now states the instant dirty-tree failure per arrival, the rewritten report, and strand removal with the fixer live, and marks the outcome as accepted because D1 makes the trigger a one-hour hang.
  The matching Open risk now states that, with the ceiling ignored, a fixer whose gate runs over ten minutes ends the run stuck within a few Merriam turns and is removed mid-edit.
- F2 (MEDIUM, design), fixed as proposed.
  D9's held-call test is now "the fork transcript holds a `tool_use` with no matching `tool_result`", wherever it sits, with the cap still measured from the fork transcript's last write; this matches the acceptance criterion's "holds a tool call open".
- F3 (LOW, consistency), fixed as proposed.
  Premise verified: `TurnEndActivity.TranscriptModTime` and `AgentActivity.LastActivity` in `internal/shuttleengine/activity.go`, and `internal/shuttleengine/doc.go` line 274, define the value as the transcript's last write.
  D9 now says those docs change to the session's last activity, forks included; the Scope's package-docs bullet and the docs constraint name them.

## Disputed

None.

## Deferred

None.
