# Card 7 — claude-prober

**Edit:**
- `internal/shuttleengine/sessionsignal.go`
- `internal/shuttleengine/claudeengine/resume.go`

**Create:**
- `internal/shuttleengine/claudeengine/prober.go`
- `internal/shuttleengine/claudeengine/prober_test.go`
- `plan:internal/shuttleengine#SessionProber` -> `type SessionProber interface`

**Uses:**
- `plan:internal/shuttleengine#Liveness`
- `internal/shuttleengine/claudeengine#claudeHomeDir`
- `internal/shuttleengine/claudeengine#Claude.TurnEndActivity`

**Intent:**
Give the reducer's two provider-specific facts a capability (D3, D4), declared in `sessionsignal.go` beside the parser and implemented by Claude in `prober.go`.

`SessionProber` has two methods:
- `ProcessLiveness(sessionID string) Liveness`: Claude finds the session's registry entry under `~/.claude/sessions` (the layout `CheckResume` reads), and answers alive when the entry's pid is alive with a start time equal to its `procStart`, dead when the pid is not alive, and unproven when no entry names the session, the registry or an entry cannot be read, the start time cannot be read, or the live pid's start time differs (a reused pid).
  It never answers dead without a registry entry, so a normally exited session whose entry Claude Code removed reads unproven, as D4 accepts.
- `TurnStartInterrupt(turnStart SessionSignal) (at time.Time, interrupted bool)`: Claude reads the transcript the turn start's `UserPromptSubmit` payload names through `transcript_path`, and answers interrupted when the transcript's newest main-chain entry is a user entry whose text starts with `[Request interrupted by user`, with that entry's own timestamp.
  Any later main-chain entry means no interrupt; a missing `transcript_path`, an unreadable transcript or an unparsable entry means no interrupt.
  It reads backward from the end, like `TurnEndActivity`.

The registry scan in `resume.go` is factored so `checkResumable` and `ProcessLiveness` share one reader of registry entries, with `checkResumable`'s behavior and messages unchanged; the liveness and start-time seams stay injectable as they are there, so the tests spawn nothing.
`Claude` asserts it implements `SessionProber` at compile time.

Tests, untagged, in `prober_test.go`: a table over a temp registry directory with injected liveness and start-time functions (alive and matching, dead pid, reused pid, unreadable start time, no entry, unreadable registry, an entry for another session), and a table over temp transcripts (interrupt marker newest, a later prompt after it, a later assistant entry, a sidechain marker, missing path, unreadable file).
`resume_test.go`'s existing rows keep passing unchanged.

**ImpactSummary:** Adds an optional SessionProber capability to shuttleengine and Claude's implementation; resume.go's registry scan is shared with it without changing CheckResume's answers.
