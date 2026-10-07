# Card 4 — shuttle-agent-activity

**Create:**
- `internal/shuttleengine/activity.go`
- `internal/shuttleengine/activity_test.go`
- `internal/shuttleengine/claudeengine/activity.go`
- `internal/shuttleengine/claudeengine/activity_test.go`

**Edit:**
- `internal/shuttleengine/rundir.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/waitmark.go`
- `internal/shuttleengine/claudeengine/usage.go`

**Intent:**
D5, data side: a provider-neutral reading of a worktree's agent activity, for batten's quiet and api-error notices (card 7).

A shuttle run's liveness needs a pid, which `RunState` lacks: `RunState` gains `PID int` (`json:"pid,omitempty"`), the pid of the process that waits on the run.
`Runner.start` records `os.Getpid()` in the first persist, and `reconstructAndWait` in `attach.go` rewrites it to the attaching process's pid before it waits (a persist failure there is a `logger.Warn`, since the field is informational).
A run counts as live when its `Outcome` is `runOutcomeRunning` and its `PID` is live by the package's `isAlive` seam; a record with no pid (an older binary's) is not live.

`activity.go` declares, in `shuttleengine`:
- an optional capability beside `Engine`, `ActivityReader`, with one method taking a turn-end `Event` and returning a provider-neutral `TurnEndActivity`: the transcript's last write time (zero when unreadable), and whether the transcript's newest main-chain turn end is an API error, with its text;
- `AgentActivity`, one live run's reading: its strand name (`RunState.StrandName`, the strand guid when empty), its last activity (the newer of the transcript's last write and the events file's last write; with neither readable, the run's `CreatedAt`), and the API-error flag and text;
- `ReadAgentActivity(cfg Config, anchorPath string, engine Engine) ([]AgentActivity, error)`: scans `runDirRoot(cfg, anchorPath)` as `ReadWaitMarker` does, skips a non-directory, a missing or undecodable `run.json` (logged, as `ReadWaitMarker` logs a torn marker) and a run that is not live, parses each live run's events file past `PromptOffset` through `engine.ParseEvents`, and asks the engine's `ActivityReader` about the newest turn end (`EventStop` or `EventWaiting`); a run with no turn end yet is judged by its events file alone.
  An engine without `ActivityReader` yields readings with no transcript part. An absent root returns none and no error.

`claudeengine/activity.go` implements `ActivityReader` on `*Claude`: the transcript path from the turn end's `transcript_path` (as `ContextTokens` reads it), its modification time, and the newest main-chain (not `isSidechain`) assistant entry that ends a turn read backward with the existing `readBackward` helper; that entry is an API error when it carries Claude's `isApiErrorMessage: true` marker, and its text is the entry's final text.
The marker knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`); `shuttleengine`'s surface names no provider field.

Tests (tier 1, injected `isAlive`, temp run directories and transcript fixtures written by the test):
- `activity_test.go`: a live run with a transcript newer than its events file reports the transcript time; a dead-pid run, a pid-less record and a finished run are skipped; an unreadable transcript falls back to the events file, and a run with no events to its `CreatedAt`; an API-error turn end is reported with its text; an absent root returns none;
- `claudeengine/activity_test.go`: a transcript whose newest main-chain assistant turn end carries `isApiErrorMessage` reads as an API error with its text, one whose error is only on a sidechain entry or followed by a normal turn end does not, and a missing `transcript_path` or file reads as unreadable.
Document the reading in both packages' `doc.go`.

**ImpactSummary:** `RunState` gains one optional field written by `start` and `reconstructAndWait`; every existing reader ignores it; the new surface has no caller until card 7.
