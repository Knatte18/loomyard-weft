# Card 5 — tmuxkit-hermetic-server

**Create:**
- `internal/testkit/tmuxkit#serverConfig`
- `internal/testkit/tmuxkit#startServer`
- `internal/testkit/tmuxkit#registerKey`
- `internal/testkit/tmuxkit#registeredKeys`
- `internal/testkit/tmuxkit#registryMu`

**Edit:**
- `internal/testkit/tmuxkit#Socket`
- `internal/testkit/tmuxkit#KillOnCleanup`
- `internal/reedengine#Engine.sessionlessSocketHolderPersists`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#CleanClaudeEnv`
- `internal/agentname#StrandNameEnv`
- `internal/agentname#ParentEnv`

**Prosa:**
- `pattern/PATTERN-test-isolation.md`

**Intent:** D4: every server tmuxkit covers is pre-started, hermetic and non-login.
`serverConfig` writes, once per test binary under the kit's directory, a tmux config with `set -g default-shell <shell>`, `set -g default-command <shell>`, `set -g exit-empty off` and `set -g @lyx_test_server on`, where `<shell>` is `$SHELL` or `sh` when unset; a non-empty `default-command` is what makes tmux start a pane's shell without the login flag, so the operator's rc files still run and the login profile does not.
`startServer` runs `tmux -f <config> -L <key> start-server`, so the server never reads `~/.tmux.conf`, and `registerKey` records the key under `registryMu` for the end-of-package check card 10 adds.
The pre-started server, not reed's boot, is the process every session and pane on the key inherits its global environment from, so `startServer` applies the environment hygiene reed's own server spawn applies: its environment is the test process's passed through `reedengine.CleanClaudeEnv`, less `LYX_TRACE_ID`, `agentname.StrandNameEnv` and `agentname.ParentEnv`, which no llm-tier pane may inherit from the gate's agent shell.
Its cwd is the kit's directory.
`Socket` and `KillOnCleanup` call both before returning; a start failure is `t.Fatalf`; on a key whose server already runs, `start-server` is a client command that changes nothing.
Bound: the pre-started server keeps reed's environment hygiene but not reed's other server-start properties.
It runs without reed's `-v` debug flags and with the kit's directory as its cwd, so it writes no `tmux-server-<pid>.log` into the hub logs dir; a test that asserts on those artefacts registers its key after reed's boot instead (card 6).
A server reed starts itself carries no `-f`, so it reads the operator's `~/.tmux.conf` and its strand panes start login shells, as the decision record's parent amendment "D4 bound: a server reed boots itself after a test's own down reads the operator config" accepts.
That covers the first boot of a test that registers after the boot, and every boot after a test's own `down` or `kill-server` on its key.
Such tests pin reed's own server start, teardown or recovery, which a pre-started server would bypass.
Their key stays registered, so card 10's check passes them.
The pre-started server holds the socket with no session, and reed's `sessionlessSocketHolderPersists` treats such a holder as stale after `staleSocketGrace` and reaps it, which would cost five seconds per cold boot and replace the hermetic server with a config-less one.
The one amendment to reed, licensed by the decision record's parent amendment "D4: reed's stale-holder probe skips the hermetic test server": before the grace loop, the probe asks the holder `show-options -gqv @lyx_test_server`, and a holder that answers `on` is tmuxkit's pre-started server, never stale, so the probe returns false at once; a holder that answers anything else, or does not answer, runs today's loop unchanged.
What it lets through and its bound: the reap is skipped only for a server whose config sets `@lyx_test_server on`, which only tmuxkit's config does and reed's own boots never do; an operator config that sets that option disables the reap for its own servers, and a server that sets `exit-empty off` without it is reaped as today.
Tests: `tmuxkit_integration_test.go` gains `TestSocket_StartsHermeticNonLoginServer`, which sets `HOME` to a temp directory holding a `.tmux.conf` that sets a marker option, calls `Socket`, and asserts through `show-options -g` that the marker is unset, `default-command` is the shell, `exit-empty` is `off` and `@lyx_test_server` is `on`, then runs `new-session -d` on the key and `split-window`, and asserts the split pane's `#{pane_start_command}` is the shell, which a login-shell pane leaves empty.
It also sets `CLAUDECODE`, a `CLAUDE_CODE_`-prefixed variable, `LYX_TRACE_ID`, `LYX_STRAND_NAME` and `LYX_PARENT` before `Socket`, and asserts that `show-environment -g` on the key lists none of them.
It uses `t.Setenv` and names `HOME` and those variables as its global state.
`TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm` gains assertions that the server pid before `Up` (from `display-message -p #{pid}` on the pre-started server) equals the pid after, and that the cold boot took less than `staleSocketGrace`.
`tmuxkit`'s budget row rises by one.
`PATTERN-test-isolation.md` gains a section "Hermetic servers" stating the pre-start, the config's four options, the environment hygiene the pre-start shares with reed's spawn, that no pre-started server reads `~/.tmux.conf`, the bound above for servers reed starts itself, citing that amendment, and reed's `@lyx_test_server` probe as the one production accommodation, with its bound.
`tmuxkit`'s package comment states the pre-start with the kit's own config, and its exemption bound widens to starting servers under that config; the probe's doc comment states the early return and its bound.

**ImpactSummary:** `Socket` and `KillOnCleanup` gain a pre-start every tmux-tier test runs through, and one reed probe gains an early return that only a server carrying tmuxkit's option takes.

**Verify:** lyx gate test --tags tmux ./internal/testkit/tmuxkit ./internal/reedengine
