# Card 1 — Record SessionStart and PermissionRequest on every run, and capture the hook corpus

**Create:**
- `internal/shuttleengine/claudeengine#hookEventPermissionRequest`
- `internal/shuttleengine/claudeengine/testdata/corpus/hook-capture/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/hook-capture/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/hook-capture/transcript.jsonl`

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
Bound: both new recordings are `recordingCommand` with `alwaysSucceeds` true, as every recording hook but `Stop` is, so they exit 0 and print nothing to standard output.
The `PermissionRequest` recording therefore answers no permission request, leaving Claude's own dialog to decide it, and the `SessionStart` recording adds nothing to the session's context; only the compaction command's output reaches it, as today.
Update `buildSettings`'s doc comment, the settings.go file comment and the hook list in `claudeengine/doc.go`.
`TestBuildSettings_RecordingHooks` and `TestBuildSettings_SessionStartHook` gain rows: every run records `SessionStart` for every source, `compact` included, once each, and `PermissionRequest`, each command ending in the `; true` suffix the other recording rows assert; a run with a compaction command also runs that command on `compact` only and after the recording command.

The capture scenario is a new `llm`-tier test, `TestSmokeTurnsHookCapture`, beside `TestSmokeTurnsSessionHooks` in `internal/shuttlecli/smoke_turns_test.go`, which raises `internal/shuttlecli`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.
It drives one run's session through startup, a typed turn, a permission request, `/clear`, `/compact` and an `/exit`, then a resumed launch of the same run that is killed, so every hook line lands in the run's one events file.
For each hook line it records the hook name, the payload and its order against pane captures: the trust or bypass gate on screen, and the first `StartupReady` reading.
It records Claude's session registry entry for the session when `SessionStart(startup)` lands, after `/exit` and after the kill.
It keeps the events file, the transcript and those readings in a directory it logs, as `keepOnFailure` does for a failing scenario.

The implementer runs it once through the card's `Verify:` command and promotes the kept files to the claudeengine corpus case `testdata/corpus/hook-capture/`, a copy and a trim as `smoke_turns_test.go`'s file comment describes:
`events.jsonl` holds the payload lines, as the existing cases do, with session ids and paths scrubbed to fixed values;
`transcript.jsonl` is the trimmed transcript of the session its last turn end names;
`case.yaml` carries the `source`, `date` and `claude_code_version` fields the existing cases carry, the order of `SessionStart(startup)` against the gate and the first ready pane and each registry reading in free fields beside them, and the `gated`, `skills`, `turn_ends` and `states` keys `TestCorpus_ReplaysEachTurnEnd` reads, with `states` naming the fold's reading after each signal-bearing line today.
`TestCorpus_ReplaysEachTurnEnd` then replays the case through `ParseEvents`, the wait loop and the fold with no change to `corpus_test.go`, since its YAML decode ignores the free fields.
`StopFailure` stays on its reconstructed fixture.

`TestParseSessionSignals` gains rows that read the case's `events.jsonl` and expect today's signals for each `SessionStart` source, `UserPromptSubmit`, `Stop`, `Notification` and `SessionEnd`;
the rows for `PermissionRequest` and the mapped sources, and the `states` entries their rules change, land with the cards that map them.

Blocker clause from D1: when the corpus shows that the `--settings` file's `SessionStart(startup)` line does not land on its own, with no turn typed, the implementer records that in `case.yaml` and reports it as a blocker in its batch report;
card 7 is then not built, and every other card is.

**ImpactSummary:** every run's settings.json gains a recording `SessionStart` entry and a `PermissionRequest` entry, both silent and exiting 0; the parser skips `PermissionRequest` lines until card 4, so no reading changes yet.

**Verify:** lyx gate test --tags llm ./internal/shuttlecli -- -run TestSmokeTurnsHookCapture -count=1
