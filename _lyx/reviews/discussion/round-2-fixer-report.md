# Fixer report — discussion round 2

## Changes

- **F1** (fixed as proposed): in `_lyx/discussion/decision-record.md`, D2 now says an assistant entry whose stop reason is absent or null, or any value other than `tool_use`, `end_turn` or `stop_sequence`, reads unknown, and that a message cut off mid-stream leaves a null stop reason.
  D2's split-message sentence and the Auto-mode assumption are qualified to a completed message.
  The `claudeengine` Acceptance criterion gains two rows: an assistant line with a null stop reason and one with an unlisted stop reason such as `max_tokens`, both reading unknown.
- **F2** (fixed as proposed, choosing the "audit reads the recorded authorization" branch): D3 now decides that the fork audit at finalize reads the recorded authorization instead of the attaching Call's spec.
  Premise checked: `Run.finalize` (`internal/shuttleengine/wait.go`) gates `AuditForks` on `run.spec.ForkSubagents`, and `attach` (`internal/shuttleengine/attach.go`) sets `spec` to the attaching Call's normalized spec.
  Scope In's `RunState` bullet names both readers, and Acceptance gains a row: a twill-spawned run attached by a Call without `ForkSubagents` is still fork-audited when Done, and a single-spawned run attached by a Call with it is not.

## Disputed

None.

## Deferred

None.
