# Card 4 — daemon-engines-use-told-tmux

**Edit:**
- `internal/reedcli#enterSession`
- `internal/reedcli/doc.go`

**Create:**
- `internal/reedcli/entersession_integration_test.go`

**Intent:**
Decision D3: the daemon's per-session engines drive the tmux binary the daemon was told.
`enterSession` still loads the worktree's `reed.yaml` through `reedengine.LoadConfig` for every other key, then sets `cfg.Tmux = tmuxPath` before `reedengine.New(cfg, geom)`, so each watcher's resize round trips and name-repair passes go through the daemon's `--tmux` and a `--tmux` shim counts every subcommand.
Bound, stated in the function's doc comment: the daemon's `--tmux` is the `tmux` value of the worktree whose spawn started it, and its watchers ignore every other worktree's `tmux` value, a debugging wrapper included; nothing compares the two, since a config value and a resolved path to one binary differ as text and a comparison would warn on every healthy hub.
This is the daemon deriving nothing, per `PATTERN-told-geometry`.

`internal/reedcli/doc.go` adds one sentence: each watched session's engine drives the daemon's told tmux binary, whatever that worktree's config names.

Test: `entersession_integration_test.go`, tagged `//go:build integration`, is the package's first tier-2 file; it needs git and a hub fixture and no tmux server.
`TestEnterSession_DrivesTheToldTmux` builds a hub through `hubforge.CopyHub`, loads the prime worktree's resolved config, sets its `Tmux` to a path under `t.TempDir()` that exists on no `PATH` and its `Watchdog` to `off`, and writes the whole config back to the worktree's `_lyx/config/reed.yaml` through `yaml.Marshal`, the way the `ReEntryReReadsFlippedConfig` step of `TestWatchdogDaemon` seeds a flipped value.
It then calls `enterSession(h.Path, told, reedengine.SessionName(prime))` with a second distinct told path and asserts `ws.eng.TmuxPath()` equals the told path exactly and `ws.cancel` is nil.
With `watchdog: off` the call starts no goroutine, so nothing in the test reaches tmux, which keeps the file in tier 2.
The package's `TestMain` already runs through `tmuxkit.Main`.
`internal/reedcli` holds one test fewer than its budget row, so the row stays at 38.

**ImpactSummary:** One assignment in the daemon's entry path; `reedengine` and every reed verb are untouched, and only a hub whose worktrees name different tmux binaries sees a different binary spawned by the daemon's watchers.
