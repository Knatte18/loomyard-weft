# Orch watcher cycles on a stale reading after /compact

`orch-compact-stale-reading` · Notes · bug

On 2026-10-04 the operator ran `/compact` in the orch session at 10:09:49Z.
It finished at 10:11:15Z (`compact_boundary`, trigger manual, preTokens 354815, postTokens 13673).
At 10:14:53Z the watcher sent the soft-cycle stencil and then cleared and resumed the session from a handoff, although the context held about 14k tokens.
`state.json` afterwards shows `cycle_trigger: soft`.

Cause: the watcher updates `LastContextTokens` only at a turn end (`internal/orchengine/watcher.go`, the `ContextTokens(ev)` call per turn-end event).
A `/compact` produces no turn end, so the reading stayed at the pre-compaction 354815, above `soft_threshold_tokens` (300000), and the soft trigger fired once `soft_idle_s` had passed.
`transcriptContextTokens` in `internal/shuttleengine/claudeengine/usage.go` would return the same stale number even if re-read, since it takes the last main-chain assistant entry's usage, which is still the pre-compaction one.

Expected: a manual compaction resets the reading, so the operator's `/compact` is not followed by a cycle.
Options: treat a `compact_boundary` entry newer than the last assistant entry as the reading (its `postTokens`), and re-read the transcript before firing a soft or hard trigger rather than trusting the stored turn-end reading.
The same applies to Claude Code's own auto-compaction (a residual named in `internal/orchengine/doc.go`).


From GitHub #356 (intake 2026-10-04).
