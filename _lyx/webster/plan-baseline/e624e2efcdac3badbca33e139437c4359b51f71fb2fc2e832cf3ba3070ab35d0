# Card 1 — Bash timeout ceiling and the gate rule

**Create:**
- `plan:internal/shuttleengine/claudeengine#bashMaxTimeoutEnvKey` -> `const bashMaxTimeoutEnvKey = "BASH_MAX_TIMEOUT_MS"`
- `plan:internal/shuttleengine/claudeengine#bashMaxTimeout` -> `const bashMaxTimeout = time.Hour`
- `plan:internal/shuttleengine/claudeengine#bashMaxTimeoutMS` -> `var bashMaxTimeoutMS = strconv.FormatInt(bashMaxTimeout.Milliseconds(), 10)`

**Edit:**
- `internal/shuttleengine/claudeengine#buildLaunchCmd`
- `internal/shuttleengine/claudeengine#buildResumeCmd`
- `internal/shuttleengine/claudeengine#steerRawGoDeny`
- `internal/shuttleengine/claudeengine#noticeRawGoDeny`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/shuttleengine/claudeengine/settings_test.go`

**Uses:**
- `internal/shuttleengine/claudeengine#promptCacheTTLEnvKey`

**Intent:**
Decision record D1 and D2: every lyx-spawned claude session gets Claude Code's Bash timeout ceiling raised to one hour, and the raw-go steer becomes the one statement of the gate rule, telling the session to run gates and quick checks in the foreground.

In `command.go`, beside `promptCacheTTLEnvKey`, declare the three new members.
`bashMaxTimeout` is the one constant (one hour) the pane value, the steer and card 2's fork-activity cap all derive from;
`bashMaxTimeoutMS` is its decimal millisecond string (`3600000`), so the number is never spelled twice.
Their doc comments say the variable rides the pane line for the same reason `forkSubagentEnvKey` does (the reed server's environment is scrubbed of Claude Code settings), that Claude Code reads it as the ceiling on the Bash tool's `timeout` parameter, and that `BASH_DEFAULT_TIMEOUT_MS` is left at Claude Code's default, so only a call that sets an explicit `timeout` runs past the default.

`buildLaunchCmd` and `buildResumeCmd` each wrap the line with `sh.WithEnv(bashMaxTimeoutEnvKey, bashMaxTimeoutMS, cmd)` right after the `promptCacheTTLEnvKey` wrap and before the `envFileKey` wrap, so the assignment sits between `CLAUDE_ENV_FILE` and `CLAUDE_CODE_PROMPT_CACHE_TTL`.
Neither signature changes.
Each doc comment gains one sentence: the line carries the ceiling assignment, and a resume line carries it because the resumed claude inherits nothing from the launch.

In `settings.go`, `steerRawGoDeny` and `noticeRawGoDeny` become package `var`s, since the steer now concatenates `bashMaxTimeoutMS`;
the init check over `standingDenies` keeps guarding them.
The steer reads exactly:
`"raw module-wide, tmux-tier and llm-tier go runs are refused here. Gate rule: run lyx gate test in the foreground with the Bash tool timeout set to " + bashMaxTimeoutMS + ", never in the background, for example lyx gate test ./internal/x, and use its --tags flag for a tag. Run every other check in the foreground too, and never cd into the directory you already work in. The slot, strand and prebuilt-binary environment variables belong to lyx, and a command naming any of them is refused"`.
It drops "as a background Bash call" and holds no quote or backslash.
`noticeRawGoDeny` stays the steer plus a period.
The steer's own doc comment says it is the gate rule every lyx-spawned session and its forks read, which stencils point at as "the gate rule in your system prompt".

`doc.go`: the launch-line paragraph names `BASH_MAX_TIMEOUT_MS`, its one-hour value from `bashMaxTimeout`, and its bound: only a call with an explicit `timeout` runs past Claude Code's default, no call stays in the foreground past the ceiling, and at the ceiling the Bash tool ends the call or moves it to the background.
The raw-go paragraph's "The steer names `lyx gate test` as the route, run as a background Bash call." becomes: the steer is the gate rule, naming `lyx gate test` run in the foreground with the Bash tool's `timeout` at the ceiling, every other check in the foreground, and no `cd` into the session's own working directory.

Tests:
`command_test.go`'s `pwshEnvLead` and `posixEnvLead` gain the assignment in its dialect between the two existing ones (`$env:BASH_MAX_TIMEOUT_MS = '3600000'; ` and `BASH_MAX_TIMEOUT_MS='3600000' `), the comment above them says so, and the two literal lines that spell the lead out in full (the `1h` launch row in `TestBuildLaunchCmd` and the `prompt_cache_ttl_one_hour` row in `TestBuildResumeCmd`) gain it too.
`TestBuildDenyNotice_SentenceContent` asserts the non-fork notice contains `Gate rule: run lyx gate test in the foreground with the Bash tool timeout set to 3600000, never in the background`, contains `never cd into the directory you already work in`, and does not contain `background Bash call`.
No other test asserts the old steer text: the remaining uses reference the identifier.

**ImpactSummary:** every claude launch and resume line gains one environment assignment, and the deny notice and raw-go hook reason carry the new gate-rule wording; no signature changes.

**Commit:** 1: claudeengine raises the Bash timeout ceiling and states the foreground gate rule
