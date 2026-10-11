# Card 1 — Record SessionStart and PermissionRequest on every run, and capture the hook corpus

**Create:**
- `plan:internal/shuttleengine/claudeengine#hookEventPermissionRequest` -> `const hookEventPermissionRequest = "PermissionRequest"`
- `internal/shuttleengine/claudeengine/testdata/hooks/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/hooks/capture.yaml`

**Edit:**
- `internal/shuttleengine/claudeengine#settingsHooks`
- `internal/shuttleengine/claudeengine#buildSettings`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/signals_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttlecli/smoke_turns_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D2 and the recording half of D5, then D1's capture, so the corpus every later card tests against is real.

`buildSettings` installs a recording `SessionStart` hook on every run, for every source, and a recording `PermissionRequest` hook on every run.
`settingsHooks` gains a `PermissionRequest` field, and its `SessionStart` field is always filled (drop `omitempty`).
Each `SessionStart` line is recorded exactly once whatever the source.
A run with a context-after-compaction command keeps that command on the `compact` source only, run after the recording command in the same hook entry, as today;
a run without one gets a single recording entry with no matcher.
With a command, the recording for the other sources sits in an entry whose matcher names every source but `compact` (for example `startup|resume|clear`), so no source records twice.
Update `buildSettings`'s doc comment, the settings.go file comment and the hook list in `claudeengine/doc.go`.
`TestBuildSettings_RecordingHooks` and `TestBuildSettings_SessionStartHook` gain rows: every run records `SessionStart` for every source, `compact` included, once each, and `PermissionRequest`; a run with a compaction command also runs that command on `compact` only and after the recording command.

The capture scenario is a new `llm`-tier test, `TestSmokeTurnsHookCapture`, beside `TestSmokeTurnsSessionHooks` in `internal/shuttlecli/smoke_turns_test.go`, which raises `internal/shuttlecli`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.
It drives real sessions through startup, a typed turn, a permission request, `/clear`, `/compact`, a resume, an `/exit` and a killed process.
For each hook line it records the hook name, the payload and its order against pane captures: the trust or bypass gate on screen, and the first `StartupReady` reading.
It records Claude's session registry entry for the session when `SessionStart(startup)` lands, after `/exit` and after the kill.
The implementer runs it once through the card's `Verify:` command and trims the captured lines into `testdata/hooks/events.jsonl` (stamp and payload lines, one line per hook, session ids and paths scrubbed to fixed values) and `testdata/hooks/capture.yaml` (the order of `SessionStart(startup)` against the gate and the first ready pane, and each registry reading).
`StopFailure` stays on its reconstructed fixture.

`TestParseSessionSignals` gains rows that read `testdata/hooks/events.jsonl` and expect today's signals for each `SessionStart` source, `UserPromptSubmit`, `Stop`, `Notification` and `SessionEnd`;
the rows for `PermissionRequest` and the mapped sources land with the cards that map them.

Blocker clause from D1: when the corpus shows that the `--settings` file's `SessionStart(startup)` line does not land on its own, with no turn typed, the implementer records that in `capture.yaml` and reports it as a blocker in its batch report;
card 6 is then not built, and every other card is.

**ImpactSummary:** every run's settings.json gains a recording `SessionStart` entry and a `PermissionRequest` entry; the parser skips `PermissionRequest` lines until card 3, so no reading changes yet.

**Verify:** lyx gate test --tags llm ./internal/shuttlecli -- -run TestSmokeTurnsHookCapture -count=1
