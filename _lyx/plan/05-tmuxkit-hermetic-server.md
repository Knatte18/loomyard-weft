# Card 5 — tmuxkit-hermetic-server

**Create:**
- `plan:internal/testkit/tmuxkit#serverConfig` -> `func serverConfig() (string, error)`
- `plan:internal/testkit/tmuxkit#startServer` -> `func startServer(tmux, key string) error`
- `plan:internal/testkit/tmuxkit#registerKey` -> `func registerKey(key string)`
- `plan:internal/testkit/tmuxkit#registeredKeys` -> `var registeredKeys map[string]bool`
- `plan:internal/testkit/tmuxkit#registryMu` -> `var registryMu sync.Mutex`

**Edit:**
- `internal/testkit/tmuxkit#Socket`
- `internal/testkit/tmuxkit#KillOnCleanup`
- `internal/reedengine#Engine.sessionlessSocketHolderPersists`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `pattern/PATTERN-test-isolation.md`

**Intent:** D4: every server tmuxkit covers is pre-started, hermetic and non-login.
`serverConfig` writes, once per test binary under the kit's directory, a tmux config with `set -g default-shell <shell>`, `set -g default-command <shell>` and `set -g exit-empty off`, where `<shell>` is `$SHELL` or `sh` when unset; a non-empty `default-command` is what makes tmux start a pane's shell without the login flag, so the operator's rc files still run and the login profile does not.
`startServer` runs `tmux -f <config> -L <key> start-server`, so the server never reads `~/.tmux.conf`, and `registerKey` records the key under `registryMu` for the end-of-package check card 10 adds.
`Socket` and `KillOnCleanup` call both before returning; a start failure is `t.Fatalf`.
The pre-started server holds the socket with no session, and reed's `sessionlessSocketHolderPersists` treats such a holder as stale after `staleSocketGrace` and reaps it, which would cost five seconds per cold boot and replace the hermetic server with a config-less one.
The one amendment to reed: before the grace loop, the probe asks the holder `show-options -gv exit-empty`, and a holder that answers `off` is a server kept alive by design, never stale, so the probe returns false at once; a holder that answers anything else, or does not answer, runs today's loop unchanged.
Production servers never set `exit-empty`, so no production boot takes the new branch; this is the reason D4's "Reed is unchanged" is read as "reed's pane start mode is unchanged", recorded in the overview's Shared Decisions.
Tests: `tmuxkit_integration_test.go` gains `TestSocket_StartsHermeticNonLoginServer`, which sets `HOME` to a temp directory holding a `.tmux.conf` that sets a marker option, calls `Socket`, and asserts through `show-options -g` that the marker is unset, `default-command` is the shell and `exit-empty` is `off`, then runs `new-session -d` on the key and `split-window`, and asserts the split pane's `#{pane_start_command}` is the shell, which a login-shell pane leaves empty; it uses `t.Setenv` and names `HOME` as its global state.
`TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm` gains assertions that the server pid before `Up` (from `display-message -p #{pid}` on the pre-started server) equals the pid after, and that the cold boot took less than `staleSocketGrace`.
`tmuxkit`'s budget row rises by one.
`PATTERN-test-isolation.md` gains a section "Hermetic servers" stating the pre-start, the config's three options, that no test server reads `~/.tmux.conf`, and reed's `exit-empty` probe as the one production accommodation.

**ImpactSummary:** `Socket` and `KillOnCleanup` gain a pre-start every tmux-tier test runs through, and one reed probe gains an early return that production boots never take.

**Verify:** lyx gate test --tags tmux ./internal/testkit/tmuxkit ./internal/reedengine
