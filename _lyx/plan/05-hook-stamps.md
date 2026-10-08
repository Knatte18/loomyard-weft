# Card 5 — hook-stamps

**Edit:**
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/hookcmd_integration_test.go`
- `internal/shuttleengine/claudeengine/events.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine/signals.go`

**Intent:**
Install the new recording hooks and the stamp lines (D1) in `buildSettings`.

`settingsHooks` gains `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd`, each one entry with no matcher, installed for every session.
Every recording hook appends two lines to the events file: first the stamp line by one `printf` whose format is card 3's stamp shape, the hook event name a literal and the time `date -u +%Y-%m-%dT%H:%M:%SZ` passed as a `%s` argument; then the payload by today's command, `cat >> … && printf '\n' >> …`, unchanged.
The two are joined by `;`, so a failed stamp never blocks the payload append.
The four new hooks end in `; true`, so they exit 0 whatever the append does; the Stop hook and the interactive AskUserQuestion record keep today's exit status, the payload append's.
No recording hook prints to standard output.
The commands use POSIX `sh`, `date`, `cat` and `printf` only, runnable under git-bash, with no `jq` and no `lyx`.
The stamp's hook name literals and line shape come from the constants card 3 declared in `signals.go`, so writer and parser share one spelling.
The deny hooks (card 1's table) are untouched.

Tests:
- `settings_test.go`: the document carries the four new hook entries with the exact commands, Stop's and the ask record's commands are the stamp printf, `;`, then today's command byte-identical, and existing rows gain the new entries.
- `hookcmd_integration_test.go` (`integration`): run under `sh` with a payload on standard input, the Stop and the ask record commands append a stamp line naming their hook and then a 20000-byte payload byte-identical to the input, and `ParseEvents` yields the same events from the file as from the payload alone, `Raw` included; `ParseSessionSignals` pairs the payload with the stamp's time; with `date` made to fail (a `PATH` whose `date` exits 1 beside the real `cat`), the payload is still appended and its signal has a zero `At`; each new hook exits 0 and Stop exits 1 when the events file's directory is missing; no recording hook writes to standard output.

Docs: `settings.go`'s and `events.go`'s file comments name the new hooks and the stamp lines `ParseEvents` skips; `claudeengine/doc.go` describes the events file's line format, the stamp pairing and the accepted append races D1 lists.

**ImpactSummary:** Every lyx-spawned Claude session's settings.json gains four recording hooks and a stamp line before each recording hook's payload; Stop and ask payload lines and ParseEvents' events are unchanged.
