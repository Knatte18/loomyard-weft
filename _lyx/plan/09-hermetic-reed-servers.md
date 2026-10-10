# Card 9 — hermetic-reed-servers

**Create:**
- `internal/reedengine#Engine.serverSpawnArgv`
- `internal/reedengine#resolveShellPath`
- `internal/reedengine#shellOptionArgvs`
- `internal/reedengine/serverconfig_integration_test.go`

**Edit:**
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#Engine.pinGeometryOptionsLocked` -> `func (e *Engine) pinGeometryOptionsLocked(target, shellPath string)`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#Engine.probeCapabilityLocked`
- `internal/reedengine#TmuxCmd`
- `internal/reedengine#TmuxCmd.serverArgs`
- `internal/testkit/tmuxkit/tmuxkit.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/revive_test.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/attach_test.go`
- `internal/reedengine/doc.go`
- `pattern/PATTERN-test-isolation.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/testkit/tmuxkit#ConfigText`

**Intent:**
Decision D7: reed's own servers read no `~/.tmux.conf`, and strand panes start non-login shells.

`serverSpawnArgv` is the pure argv of the server-starting invocation, lifted out of `ensureServerAndSessionLocked`'s `spawnSession` closure: the debug flags, then on every GOOS but `windows` `-f /dev/null`, then `-L`, the socket, `new-session -d -s <session> -c <PaneCwd> -x <Width> -y <Height>` and the configured shell as the first pane's trailing command, as today.
`-f /dev/null` rides every spawn, since a running server ignores it.
Bound, in `doc.go`: the skip covers the operator's whole `~/.tmux.conf` on reed's per-hub servers only, every option, key binding and hook in it, and leaves the operator's own servers alone; it takes effect at a hub server's next start, so a server an older lyx started keeps the operator's config until it exits; reed sets every option it depends on itself; no `reed.yaml` key carries an operator tmux setting, and an operator who wants one sources it by hand into the running server, where it lasts until that server exits.

The capability probe is the other invocation that can start a server on reed's socket: `list-commands` is answered by a server, and with none up tmux starts one that reads the operator's config.
A config that creates a session keeps that server alive, the stale-holder reap never touches a server that lists a session, and the boot's `new-session` would then join a server carrying the operator's whole config.
So `TmuxCmd` gains an unexported `configFile` field, and `serverArgs` leads with `-f <configFile>` when it is set; `probeCapabilityLocked` runs the probe through a copy of the engine's `TmuxCmd` with `configFile` set to `/dev/null` on every GOOS but `windows`, and through the engine's own on Windows.
Bound: the flag reaches the probe's two invocations only, every other engine call keeps its argv, and against a running server it changes nothing.
The test seam is unchanged, since `execHook` receives the argv without the server flags, so the fake tmux and `TestProbeCapabilityLocked_GoesThroughTheSocketScopedTmuxCmd` see the same calls.
`probeCapabilityLocked`'s doc comment, which says the transient server exits as soon as it has answered, states that the transient server reads no operator config, so nothing the operator's config does can keep it alive.

`resolveShellPath` runs `exec.LookPath` on the configured shell and returns its absolute path, or an error naming the `shell` key and the value.
`shellOptionArgvs` returns the two session-scoped pins, `set-option -t <target> default-shell <shellPath>` and `set-option -t <target> default-command <shellPath>`, never with `-g`, because one server serves every worktree of the hub; it returns nothing for an empty `shellPath`.
`pinGeometryOptionsLocked` takes the resolved shell as its second parameter and issues `shellOptionArgvs` after its existing option pins, each non-fatal like them; the empty string skips the two pins.
The session's non-empty `default-command` is what makes tmux start a strand pane's shell without the login flag, so strand panes keep their split with no trailing command and now run their rc files and not the login profile; the first pane and Selvage keep their explicit trailing shell command, and `panebin.go` is untouched.

The boot path: `ensureServerAndSessionLocked` resolves the shell in its pre-tmux block, after card 7's socket check and before the capability probe, on every GOOS but `windows`, and refuses an unresolvable shell; it passes the resolved path to `pinGeometryOptionsLocked` after the boot and spawns through `serverSpawnArgv(runtime.GOOS, debugArgs)`.
On Windows it passes the empty string and keeps its current boot, recorded as a bound.
The attach pre-flight: `AttachArgv` resolves the shell the same way before its `pinGeometryOptionsLocked` call; an unresolvable shell logs a `logger.Warn` naming the key and passes the empty string, leaving the session's earlier values, as the pre-flight's other pins degrade, and the chain is still built; on Windows it passes the empty string.

The kit: card 8 put the kit's config text behind `ConfigText`; the kit's server keeps `exit-empty off` and the `@lyx_test_server` marker, which reed's own boots never set.
`tmuxkit.go`'s package doc sentence saying reed's own server carries no such config and starts login-shell panes becomes: a server reed starts itself reads no `~/.tmux.conf` either and pins the same two shell options per session, so the kit's server differs from reed's own only by the two extra lines.
`pattern/PATTERN-test-isolation.md`'s last "Hermetic servers" bullet, the bound that a server reed starts carries no `-f` and starts login-shell panes, is replaced by the new rule, keeping the sentence about the `-v` flags and the `tmux-server-<pid>.log` artefacts a test registers its key after reed's boot for.
`doc.go`'s Selvage paragraph, which says Selvage and the first pane carry the shell as a trailing argument, gains the strand-pane rule above, and its multiplexer-contract paragraph gains the probe's `-f /dev/null` with its bound.

Tests, untagged: `TestUp_BootValidation` gains the row `ShellUnresolvable`, whose config names a shell under `t.TempDir()` that does not exist and expects the refusal naming `shell` with zero tmux calls, skipped on Windows; the fixture's existing `does-not-exist-shell.exe` means the row `ValidWatchdog_On` and its sibling now hit this refusal on POSIX, so they set `cfg.Shell` to a shell `exec.LookPath` finds, `sh`, or keep asserting only that the error is not the watchdog one.
`revive_test.go`'s `reviveTestEngine` builds on the same fixture, and every row of `TestEnsureServerAndSession_RevivalTrigger` calls `ensureServerAndSessionLocked` directly, so on POSIX each would meet the shell refusal before the revival sentinel, the spawn or the live session it asserts; `reviveTestEngine` sets `cfg.Shell` to `sh` on POSIX, which no assertion of `TestRevive_CreatesNothingWithoutThePredicate` reads.
A sweep of the untagged test files found `TestUp_BootValidation` and `TestEnsureServerAndSession_RevivalTrigger` as the only tests that reach the boot's pre-tmux block on the shared fixture; every tagged caller boots against a real shell.
A new `TestServerSpawnArgv` in `lifecycle_test.go` pins the argv for `linux` with and without debug flags, `-f /dev/null` ahead of `-L`, and for `windows` without it.
`TestPinGeometryOptionsLocked` gains the step `ShellPinsFollowTheOptionPins`: with a path, the two session-scoped pins follow the window-size pin, carry no `-g`, and name the path; with the empty string, no `default-shell` or `default-command` call appears; its `AllOptionsIssued` and `OneOptionFailureDoesNotStopTheRest` steps pass the empty string and keep their counts.
`attach_test.go`: `newAttachTestEngine`'s fixture shell does not exist, so every `TestAttachArgv_ChainGate` row already runs the unresolvable-shell pre-flight; the shared `assertStrandOptionsReasserted` check every row makes also asserts that no `default-shell` or `default-command` set-option was issued, which with `Status_On_DoesNotSuppress`'s chained argv pins the unresolvable shell pinning nothing and still chaining, and no row is added.
`TestAttachArgv_PreflightOnAKnownGoodSession`'s set-option count moves by the two shell pins where the fixture's shell resolves, so the fixture sets a shell `LookPath` finds on POSIX and the count is written as the old count plus `len(shellOptionArgvs(...))` on POSIX and the old count on Windows.

Test, `tmux` tier, in the new `serverconfig_integration_test.go` tagged `//go:build tmux && !windows`: `TestBoot_ReadsNoOperatorConfigAndStartsNonLoginPanes` builds an engine the way `newIntegrationEngine` in `mouse_boot_integration_test.go` does, sets `HOME` with `t.Setenv` to a directory whose `.tmux.conf` sets a marker user option and creates a detached session of its own, boots through `Up`, and only then registers its key through `tmuxkit.KillOnCleanup`, so reed starts the server itself; it asserts the marker reads back empty with `show-options -gqv`, the session's `default-shell` and `default-command` read back as the resolved shell, and a `split-window -d -P -F '#{pane_start_command}'` into the session prints the resolved shell, which a login-shell pane leaves empty.
The config's own session is what makes the marker cover the probe as well as the spawn: a probe server that read the config would outlive the probe holding that session, and reed's `new-session` would join it with the marker set.
A second step, `KitConfigSharesReedsShellPins`, parses `tmuxkit.ConfigText(shell)` and asserts that the option names it sets to the shell are exactly the names `shellOptionArgvs` pins, with the kit's two extra lines being `exit-empty` and `@lyx_test_server`; this is the pin the kit cannot hold itself, since it cannot import `reedengine`.
The file references a `tmuxkit` export, so it is constrained to the `tmux` tier, per `PATTERN-test-speed`.
The `internal/reedengine` row in `test-budget.yaml` rises by two, to 246; `internal/testkit/tmuxkit` adds no test.

The new refusal and the pins are reed's, outside `PATTERN-refusal-way-forward`'s packages, and still name their way forward and are reached by a test.
`PATTERN-shell-mechanics-seam` is untouched: the resolved shell is a tmux option value, never a pane command string.

**ImpactSummary:** The server spawn argv, the probe's server flags, the session pins and the attach pre-flight in `reedengine` change, with one re-signed unexported method whose two callers and one test file are on this card; every other `TmuxCmd` caller keeps its argv; every hub server started by this binary, probe-started ones included, stops reading the operator's config, which an operator who tuned reed through `~/.tmux.conf` notices.
