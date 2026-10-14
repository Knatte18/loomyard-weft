# Card 6 — sleep-deny

**Edit:**
- `internal/shuttleengine/claudeengine#standingDenies`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildSettings_AgentAndBashHooks`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_SentenceContent`
- `internal/shuttleengine/claudeengine/hookcmd_integration_test.go`
- `cmd/lyx#deniedRecoveryForms`
- `pattern/PATTERN-no-denied-recovery.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shuttleengine/claudeengine#steerSleepDeny`
- `internal/shuttleengine/claudeengine#noticeSleepDeny`
- `internal/shuttleengine/claudeengine#sleepCommandPattern`
- `internal/shuttleengine/claudeengine#TestSleepDenyHook_DeniesMachineSleepVerbsOnly`

**Uses:**
- `internal/shuttleengine/claudeengine#commandPositionPattern`
- `internal/shuttleengine/claudeengine#denyHookCommand`
- `internal/shuttleengine/claudeengine#preToolUseOutput`

**Intent:**
Deny every lyx-spawned session the machine sleep commands, so no agent, the orch included, hibernates the machine under its own call (Decision 10).
The declaration heads above carry placeholder values; the card writes the real ones.

`steerSleepDeny` reads `the machine sleep commands are refused here; a hibernate goes through lyx hibernate request, which waits until every session is paused, and a pair agent asks its parent session to run it`, with no single quote, double quote or backslash (the existing `TestBuildSettings_NoForbiddenCharsInSteerText` checks every row).
`noticeSleepDeny` is `steerSleepDeny + "."`.
`sleepCommandPattern` is `commandPositionPattern` followed by `systemctl`, then any flag words (each whitespace, then `-` and characters up to the next whitespace or separator), then whitespace and one of `sleep`, `hibernate`, `suspend`, `hybrid-sleep` or `suspend-then-hibernate` as a whole word, ending at a character outside `[A-Za-z0-9_.-]` or the end, so `hibernate.target` is no verb.
Through the shared prefix it matches `systemctl` path-prefixed, after a separator and after `env`, `exec`, `xargs`, `sudo` or `time`; `systemctl status` and every other verb pass, since a non-flag word ends the match.
`standingDenies` gains the row, installed for every run with no config key, matcher `Bash`, command `grep -Eq '<sleepCommandPattern>' && echo '<denyJSON(steerSleepDeny)>'; true`, with the comment that it is a guardrail, not a barrier: `bash -c`, `eval`, a shell variable, `systemctl start` or `isolate` of a sleep target and other sleep routes such as `busctl`, `dbus-send` and `rtcwake` pass, and a matching spelling inside a quoted argument or another payload field is falsely denied.
Its notice joins the one-line deny notice through the shared table, so `buildDenyNotice` needs no edit.
Bound: a guardrail, not a barrier, as above; the deny is Claude-engine settings only.

Tests:
- `TestSleepDenyHook_DeniesMachineSleepVerbsOnly` (in `hookcmd_integration_test.go`, build tag `integration`, `t.Parallel`, the shape of `TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly` over `denyHookCommand(t, steerSleepDeny)` and `preToolUseOutput`) has denied rows for `systemctl` with each of `sleep`, `hibernate`, `suspend`, `hybrid-sleep` and `suspend-then-hibernate`, for `systemctl --no-wall hibernate`, `/usr/bin/systemctl hibernate`, `sudo systemctl suspend` and `cd x && systemctl hibernate`, each printing `denyJSON(steerSleepDeny)` and a newline, and passing rows for `systemctl status`, `systemctl status hibernate.target` and `systemctl restart sshd`, each printing nothing.
- `TestBuildSettings_DenyToggleMatrix` checks the sleep row installed under every input, as it does the raw go, in-place edit and computed-rm rows, and its no-denies count of PreToolUse entries rises from 3 to 4, with its message naming the sleep deny.
- `TestBuildSettings_AgentAndBashHooks` sorts the sleep row's command into its own list, wants exactly one, and keeps it out of the fork-guard list, whose `default` branch would otherwise catch it; its doc comment and in-body comment naming the always-installed Bash rows gain the sleep deny.
- `hookcmd_integration_test.go`'s file header, which lists the denies the file runs, gains the sleep deny.
- `TestBuildDenyNotice_SentenceContent` wants `noticeSleepDeny` in the autonomous, interactive and fork notices, beside the in-place edit and computed-rm sentences; `TestBuildDenyNotice_MatchesInstalledDenies` covers the new row through the table unchanged.

`cmd/lyx#deniedRecoveryForms` gains the token sequences `systemctl sleep`, `systemctl hibernate`, `systemctl suspend`, `systemctl hybrid-sleep` and `systemctl suspend-then-hibernate`, since the list follows claudeengine's deny table, and `PATTERN-no-denied-recovery.md`'s closed-list sentence names them; the flag-word spellings stay pattern-shaped outside the closed list, as the in-place spellings do.
No scanned file spells one today, and card 5 keeps its spec rows and docs free of them.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengine/claudeengine` from 111 to 112.

**ImpactSummary:** every Claude session lyx spawns gains one standing Bash deny and one notice sentence; the four settings tests that enumerate the always-installed rows, and the no-denied-recovery forms, follow it.

**Commit:** 6: claudeengine: deny the machine sleep commands to spawned sessions
