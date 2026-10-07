# Card 10 — reed-window-op

**Create:**
- `internal/reedengine/window.go`
- `internal/reedengine/window_test.go`
- `internal/reedengine/window_integration_test.go`

**Edit:**
- `internal/reedengine/probe.go`
- `internal/reedengine/probe_test.go`
- `internal/reedengine/panebin.go`
- `internal/reedengine/panebin_enforcement_test.go`
- `internal/reedengine/faketmux_test.go`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/shell/shell.go`
- `internal/logger/logger.go`

**Intent:**
D7, reed op: a new `Engine` method opens a detached, named window in this worktree's reed session running a `lyx` command, for card 11's `lyx batten run <slug> --window`.

`window.go` declares the op (for example `OpenWindow(name string, lyxArgs []string) (WindowResult, error)`, the result carrying the window id, its name and whether an existing live window was found) and its sentinel errors:
- with no reed session for the worktree (`hasSession` false) it returns a no-session sentinel and creates nothing;
- a window of that exact name (`=<session>:=<name>`) whose pane is live is returned as existing and nothing is started; one whose pane is dead is killed and replaced;
- otherwise one multiplexer invocation runs `new-window -d -P -F '#{window_id}' -t =<session>: -n <name> <command>` chained with `set-option -w -t =<session>:=<name> remain-on-exit off`: detached and never selected, so it takes no focus, and the option is chained by exact name because an option chained after a detached `new-window` without a target lands on the current window, the strands' window, whose `remain-on-exit` must stay on;
- after the invocation it reads `remain-on-exit` back from the new window by the printed id; a window that does not read `off` (two concurrent calls sharing a name, where the chained option landed on the older window) is killed by that id and the op returns a read-back sentinel;
- a failed `new-window` returns a new-window-failed sentinel wrapping the multiplexer's error.
The id tmux prints is what every later op addresses the window by; the name is for display.

The command string is composed in `panebin.go` from the pane-binary prelude (`paneBinPrelude`), so `lyx` resolves to the spawning binary (`PATTERN-pane-binary-resolution`), joined with the quoted arguments through `internal/shell` (`PATTERN-shell-mechanics-seam`); no strand name exports, since the window is not a strand.
`panebin_enforcement_test.go` scans for the `new-window` literal beside `split-window`, with `window.go` allowlisted and its reason stated.
The process start is logged through `internal/logger` with the window id and command (`PATTERN-spawn-observability`).
`new-window` joins `requiredSubcommands` in `probe.go`, since the op cannot degrade, and `probe_test.go`'s fixture lists it.
Bound, in `doc.go`: the op only chooses where a command runs; the window lives as long as the reed session, and the session's other windows keep `remain-on-exit on`.

Tests:
- `window_test.go` (tier 1, the fake multiplexer extended for `new-window` and the read-back): no session returns the no-session sentinel; an existing live window is reported and nothing is created; a dead one is killed and replaced; a failed `new-window` returns its sentinel; a read-back other than `off` kills the window by id and returns the read-back sentinel; the composed command carries the prelude;
- `window_integration_test.go` (tmux tier, through the package's existing `tmuxkit.Main` and its keys): the window opens detached in the session without changing the current window, closes when its command exits (also when it exits at once), reads `remain-on-exit off` while the session's strand window still reads `on`, and a second call while it is live starts nothing.

**ImpactSummary:** one new engine op and one more required multiplexer verb; the capability probe now refuses a multiplexer without `new-window`; no existing op changes.
