# Card 14 — Claude's settings merge the Stop delivery into the turn-end record and add the idle waiter

**Create:**
- `internal/shuttleengine/claudeengine#strandFlagName`
- `internal/shuttleengine/claudeengine#creelStopHookCommand`
- `internal/shuttleengine/claudeengine#creelWaitCommand`
- `internal/shuttleengine/claudeengine#creelWaitTimeout`
- `internal/shuttleengine/claudeengine#creelWaitHookCommand`
- `internal/shuttleengine/claudeengine#TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock`

**Edit:**
- `internal/shuttleengine/claudeengine#hookCommand`
- `internal/shuttleengine/claudeengine#settingsHooks`
- `internal/shuttleengine/claudeengine#creelMailHookCommand`
- `internal/shuttleengine/claudeengine#buildSettings`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestBuildSettings_CreelMailHook`
- `internal/shuttleengine/claudeengine#TestPrepare_CreelMailHookFollowsTheBoardDirectory`
- `internal/shuttleengine/claudeengine#TestCreelMailHook_RunsLymOnlyWhenTheStrandsFlagStands`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#recordingCommand`
- `internal/shuttleengine/claudeengine#runHookCommand`
- `internal/shuttleengine/claudeengine#parseSettings`
- `internal/shuttleengine/claudeengine#hooksFor`
- `internal/shuttleengine/claudeengine#engineWith`
- `internal/shuttleengine/claudeengine#templateConfig`
- `internal/shuttleengine#Launch`

**Intent:**
D2 and D4 rendered: Claude Code runs an event's matching hooks in parallel, so a separate creel Stop hook could not keep the recording hook from logging a turn end the block then cancels, and `ParseEvents` would read every blocked Stop as a turn end; the Stop's record and its creel delivery become one command.
Only a session in a hub, told a creel flag directory, gets any of it; a standalone run keeps today's Stop recording, so `TestBuildSettings_MatchesGolden`, which renders with no flag directory, stays byte for byte.

`strandFlagName` returns the plain-`sh` fragment `creelMailHookCommand` opens with today, which forms the flag's file name in `$f$n` from `$LYM_STRAND_NAME` by parameter expansion alone; `creelMailHookCommand` uses it and is otherwise unchanged.
`creelStopHookCommand(quotedFlagDir, quotedEventsPath)` reads the hook's standard input once into a variable (`in=$(cat)`), forms the flag name, and, only when the flag stands, pipes the payload to `lym creel hook-read claude` run from `$CLAUDE_PROJECT_DIR`, writing straight to the hook's standard output; when that exits 0, a block was written and the command exits 0 having appended nothing to the events file, neither the stamp line nor the payload.
Otherwise, or when the flag is absent, it appends the stamp line `recordingCommand` writes and then the payload and a newline, and exits with the payload append's status, as the recording command does today; the stamp is written only together with the record, after the creel step decided.
Writing straight to the hook's output keeps the acknowledge-after-write order: hook-read acknowledges only after the block reached Claude Code's pipe.
Bound: a merged command killed at its timeout after the write never reaches its record step, so when Claude Code drops the killed hook's output the turn ends with no Stop record; the window is the claim's 10-second daemon call against the hook's timeout.

`hookCommand` gains `AsyncRewake bool` (JSON `asyncRewake`, omitempty) and `Timeout int` (JSON `timeout`, seconds, omitempty), so every existing hook renders unchanged.
`creelWaitHookCommand` changes to `$CLAUDE_PROJECT_DIR` and runs `creelWaitCommand`; its entry carries `AsyncRewake: true` and `Timeout: creelWaitTimeout`, 86400 seconds, well under the JavaScript timer ceiling of 2^31−1 ms, and no `rewakeSummary`, which is internal to Claude Code.
`buildSettings`, given a non-empty `creelFlagDirPosix`, renders `Stop` as two entries with no matcher, the merged command and then the waiter, beside the PostToolUse mail hook it already adds; `settingsHooks`' doc and the file's header describe the merged Stop and the waiter.
`Claude.Prepare` sets `Launch.CreelHooks` (card 2) exactly when it passes a non-empty flag directory, so a strand is marked only when its settings carry the PostToolUse, Stop and waiter hooks.

Tests:
`TestBuildSettings_CreelMailHook` also pins the `Stop` entries: in a hub the merged command and the waiter entry with `asyncRewake` true and `timeout` 86400; standalone the single recording command.
`TestPrepare_CreelMailHookFollowsTheBoardDirectory` also asserts `Launch.CreelHooks` is true in a hub and false standalone.
`TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock`, a `tier1` test in `hookcmd_integration_test.go` beside the other tests that run hook commands under `sh`, runs the merged command with a stub `lym` on `PATH` that prints a fixed block and exits as the row says, its rows:
flag standing, stub exits 0: stdout is the block, the events file is empty, exit 0;
flag standing, stub exits 1 printing nothing: stdout empty, the events file holds a stamp line naming Stop and then the payload byte for byte, which `ParseEvents` reads as the payload alone, exit 0;
flag absent: the stub is never run (it would fail the row) and the record is the same;
the stub also records the standard input it got, which must equal the payload, so the one read reached both steps;
an unwritable events file, its directory missing, with the stub exiting 1: stdout empty and a non-zero exit, as the recording command fails today.
`TestCreelMailHook_RunsLymOnlyWhenTheStrandsFlagStands` gains a row for `creelWaitHookCommand`: it runs `lym creel hook-wait claude` from the project directory whether or not a flag stands.
The claudeengine doc describes the merged Stop, the waiter and the mark.

**ImpactSummary:** the hub branch of buildSettings and Prepare's launch change; buildSettings keeps its signature, so its many test callers and the standalone golden are unaffected.
