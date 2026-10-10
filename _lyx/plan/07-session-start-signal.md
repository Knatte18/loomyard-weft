# Card 7 — session-start-signal

**Create:**
- `internal/shuttleengine#SessionSignalSessionStart`
- `internal/shuttleengine/claudeengine#hookEventSessionStart`
- `internal/shuttleengine/claudeengine#compactionMatcher`
- `internal/shuttleengine/claudeengine#SessionStartContext`

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#SessionSignal`
- `internal/shuttleengine#SessionFold`
- `internal/shuttleengine#SessionFold.applySignal`
- `internal/shuttleengine#RunSessionState`
- `internal/shuttleengine#readRunSessionState`
- `internal/shuttleengine/sessionstate_test.go`
- `internal/shuttleengine/sessionread_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine#settingsHooks`
- `internal/shuttleengine/claudeengine#buildSettings` -> `func buildSettings(eventsPathPosix string, interactive bool, cfg shuttleengine.Config, forkSubagents, allowAgentTool bool, contextAfterCompaction string) ([]byte, error)`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#signalFromPayload`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/hookcmd_integration_test.go`
- `internal/shuttleengine/claudeengine/signals_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The launch spec carries a provider-neutral context-after-compaction command, which the Claude engine renders into a `SessionStart` hook, and the hook's recording line parses to a signal the fold ignores (D11's provider half, D7's non-moving kind).
`Spec` gains `ContextAfterCompaction string`: a shell command whose standard output the provider adds to the session's context right after every compaction; empty for every run but the one that sets it, and `validate` does not inspect it.
Bound: the field is an escape hatch that runs a verbatim command inside the provider's hook shell; `orchcli` is its only setter (card 12), so only the orch run carries it, and the command runs only at a compaction of that run, never on any other session event.
`buildSettings` takes it and, when non-empty, adds a `SessionStart` entry to `settingsHooks` (`json:"SessionStart,omitempty"`) with matcher `compactionMatcher` and two commands in order: the recording command for `hookEventSessionStart` built by `recordingCommand` with `; true`, then the given command verbatim; an empty value renders no `SessionStart` key.
`Prepare` passes `spec.ContextAfterCompaction`, and the file comment of `settings.go` lists the new hook.
The package doc in `doc.go` follows: its payload-to-signal mapping gains the `SessionStart` row, its recording-hook list gains `SessionStart` for a run whose spec sets the command, and its sentence that no recording hook prints to standard output gains the exception that the session-start entry's second command prints the provider's additional-context JSON.
`SessionStartContext` renders the provider's additional-context JSON for a session-start hook, `{"hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":<text>}}`, through `encoding/json`, so the orch verb prints it without spelling the shape.
`signalFromPayload` reads a `SessionStart` payload as a `SessionSignalSessionStart` signal carrying the payload's `source` in the new `SessionSignal.Source` field.
`SessionFold.applySignal` returns at once on that kind, before `newest` is replaced, so the reading stays as it was; the fold records the newest session-start's time in a new field, which `readRunSessionState` copies to `RunSessionState.SessionStartAt` (zero when none), so a watcher can log the hook's event without parsing signals itself.
`shuttleengine/doc.go`'s session-state section adds the session-start signal to its list of signals, states that the fold does not move on it, and names `RunSessionState.SessionStartAt`.
`settings_test.go`: `TestBuildSettings_RecordingHooks` and the matrix test take the new argument with an empty command and a new test asserts the two-command `SessionStart` entry with the `compact` matcher for a set command and its absence for an empty one, plus a test of `SessionStartContext`'s exact JSON; `hookcmd_integration_test.go` takes the new argument.
`signals_test.go` gains the `SessionStart` row with its source; `sessionstate_test.go` gains a row where a session-start after a turn end leaves the state and cause unchanged; `sessionread_test.go` asserts `SessionStartAt`.

**ImpactSummary:** One new spec field, one new signal kind and one new hook entry keyed on it; `buildSettings` gains a parameter with callers in `Prepare` and two tests, and every other run renders the same settings as before.
