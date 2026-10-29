# Card 6 — publish-shell-guard

**Edit:**
- `internal/shellguard#Switch`
- `internal/shellguard#rowsAfterRawGo`
- `internal/shellguard#TestRows_PatternsDenyAndAllow`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine/claudeengine#denyInputs`
- `internal/shuttleengine/claudeengine#shellSwitchInstalled`
- `internal/shuttleengine/claudeengine#buildSettings` -> `func buildSettings(eventsPathPosix string, interactive bool, cfg shuttleengine.Config, claude Config, forkSubagents, allowAgentTool, denyPublish bool, contextAfterCompaction, creelFlagDirPosix string) ([]byte, error)`
- `internal/shuttleengine/claudeengine#buildDenyNotice` -> `func buildDenyNotice(interactive bool, cfg shuttleengine.Config, claude Config, forkSubagents, allowAgentTool, denyPublish bool) string`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#recordingHookCommands`
- `internal/shuttleengine/claudeengine#TestPrepare_AppendSystemPromptMatchesDenyNotice`
- `internal/shuttleengine/claudeengine#TestBuildSettings_PromptSuggestionOff`
- `internal/shuttleengine/claudeengine#TestBuildSettings_RecordingHooks`
- `internal/shuttleengine/claudeengine#TestBuildSettings_SessionStartHook`
- `internal/shuttleengine/claudeengine#TestBuildSettings_CreelMailHook`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildSettings_AgentAndBashHooks`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_MatchesInstalledDenies`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_SentenceContent`
- `internal/shuttleengine/claudeengine#TestBuildSettings_MatchesGolden`

**Uses:**
- `internal/shellguard#commandPositionPattern`
- `internal/shuttleengine/claudeengine#TestStandingDenies_RenderEveryShellGuardRow`
- `internal/shuttleengine/claudeengine#TestShellGuardDenies_RenderedCommandsGrepThePayload`
- `internal/shellguard/doc.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
D7: the PR-Review session's launch spec installs shell-guard denies of `git push` and `gh` for that session alone, through a new `internal/shellguard` row hung on a launch-spec switch.
`shellguard` gains `SwitchDenyPublish Switch = "deny_publish"` and, appended to `rowsAfterRawGo`, the row `publish` on that switch: one pattern, `commandPositionPattern` followed by either `git`, any flag words (`-C <dir>` and other `-x` words), whitespace and `push` as a whole word, or `gh` as a whole word; its steer `git push and gh are refused in this session: lym pushes the task branch after it gates your commits, and only lym opens, merges or closes the pull request; commit your change and tell the operator` and its notice the steer plus a period; no `Forms`, since no stencil names the denied step as a way forward.
Its row comment states the bound: a guardrail over static text, not a barrier, so `bash -c`, a script, `env` forms the pattern does not list, a `git` alias or `hub` pass, and a `gh` spelling after a separator inside a quote `commandPositionPattern` misreads is falsely denied.
`shuttleengine.Spec` gains `DenyPublish bool`, engine vocabulary that validate does not inspect.
claudeengine: `denyInputs` gains `denyPublish`; `shellSwitchInstalled` maps `SwitchDenyPublish` to `in.denyPublish`; `buildSettings` and `buildDenyNotice` take `denyPublish` after `allowAgentTool` and set it on their `denyInputs`; `Claude.Prepare` passes `spec.DenyPublish` to both.
Every other caller passes `false`, so every existing run installs and announces exactly what it does today; the listed tests change only their call sites, bar the two below.
`TestBuildSettings_DenyToggleMatrix` gains a `denyPublish` column and rows `publish_on` and `publish_off`, asserting a Bash entry carrying the publish steer exactly when the switch is on; `TestRows_PatternsDenyAndAllow` gains `publish` rows denying `git push`, `git -C x push origin b`, `gh pr merge 3` and `cd x && gh api y`, and allowing `git status`, `echo gh`, `git log --grep=push` and `ghost`.
`TestStandingDenies_RenderEveryShellGuardRow` and `TestShellGuardDenies_RenderedCommandsGrepThePayload` cover the new row through `Rows` unchanged.
The shellguard doc names the switch; the claudeengine doc's deny list names the publish deny and its switch.

**ImpactSummary:** One new opt-in shell-guard row and a launch-spec switch; the settings builders' re-sign touches only claudeengine and its tests, and every run that does not set the switch renders byte-identical settings, so the golden stays as is.

**Commit:** 6: shellguard: a publish deny of git push and gh on a launch-spec switch
