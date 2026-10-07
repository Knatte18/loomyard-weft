# Card 12 — reload-plugins-sequence

**Create:**
- `plan:internal/shuttleengine#Runner.ReloadPlugins` -> `func (r *Runner) ReloadPlugins(guid string) error`

**Edit:**
- `internal/shuttleengine#SessionCycler`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/session_test.go`

**Intent:** D10, provider half: Claude Code's `/reload-plugins` rebuilds the model-facing skill list the Skill tool reads, so a skill deployed after the orch session started becomes loadable without restarting the process (#393).
The command is provider grammar, so it lives in `claudeengine`, per `PATTERN-shuttle-provider-seam`.

Add `ReloadPluginsSequence() []PaneInput` to `SessionCycler`, documented as the key choreography that makes the live session re-read its installed plugins and skills without starting a turn.
Implement it on `*Claude` in `session.go` as `/reload-plugins` typed unsubmitted with `submitSettleMS`, then a bare `Enter` key step, with no leading Escape, the same shape Card 1 gives `ClearSessionSequence` and for the same reason.

Add `Runner.ReloadPlugins(guid)` in `internal/shuttleengine/session.go`, shaped like `Runner.ClearSession`: the told-error check, the `SessionCycler` capability check, the shuttle-strand and live-strand guards, then `playInputs` over the sequence, skipping `requireReadyAgentPane` because the caller has already probed idleness.

The fake cycler in `session_test.go` implements the new method, and the existing test pinning that a plain engine's session methods name the missing `SessionCycler` capability gains `ReloadPlugins`.
`claudeengine/session_test.go` pins the exact two-step sequence.

Assumption A1: the deployed Claude Code has `/reload-plugins`, and it ends no turn; R1 holds when it does not, and the next card's skill-load step logs an unloadable skill as `unknown`.

**ImpactSummary:** Adds one SessionCycler method with its single Claude implementation, the test fake, and one Runner method; no existing caller changes.
