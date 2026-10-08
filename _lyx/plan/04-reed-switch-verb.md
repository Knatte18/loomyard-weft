# Card 4 — `lyx reed switch --next|--prev`

**Edit:**
- `internal/reedengine/overlay.go`
- `internal/reedcli/cli.go`
- `internal/reedcli/cli_test.go`
- `internal/reedengine/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-REED-SUITE.md`

**Create:**
- `internal/reedengine/switch.go`
- `internal/reedengine/switch_test.go`
- `internal/reedengine/switch_integration_test.go`
- `internal/reedcli/switch.go`

**Intent:**
Add the verb D10 specifies, which the `M-Left`/`M-Right` bindings (card 7) call through `run-shell`: it switches one told client to the next or previous session of the server in numeric session-id order, wrapping at either end, so the order matches the bar's `#{S:}` loop (card 5) instead of `switch-client -n`'s name order.

Engine (`switch.go`): an exported, engine-less function, e.g. `SwitchClient(tmuxPath, socketPath, client string, next bool) error`, built on a `TmuxCmd` that addresses the server by socket path (`-S <path>`) instead of `-L <name>`; `overlay.go` gains that mode beside `NewTmuxCmd` (the `execHook` seam works for both).
It runs exactly three tmux commands on the told socket and nothing else:
`list-sessions -F '#{session_id} #{session_name}'`, `display-message -p -c <client> '#{session_id}'` for the client's current session, and `switch-client -c <client> -t <session id>`.
Session ids are sorted numerically (`$2` before `$10`); the target wraps; a server with one session is a no-op success that issues no `switch-client`.
A failed listing (unreachable socket), a client the server does not know, or a current session missing from the listing is an error naming what failed.
A pure helper (`nextSessionID(ids []string, current string, next bool) (string, error)`) carries the ordering so it is unit-tested without tmux.
The function reads no working directory, no reed config and no reed state (`PATTERN-cwd-resolution`, `PATTERN-told-geometry`).

CLI (`internal/reedcli/switch.go`): `switch` with flags `--next`, `--prev`, `--socket <path>`, `--client <name>`, `--tmux <path>`, a `Short`, and `clihelp.SkipStencilSeedAnnotation` (a key press must never run the root pre-run's stencil-seed pass).
Exactly one of `--next`/`--prev` is required, and `--socket`, `--client` and `--tmux` must be non-empty; a violation, and every engine error, is a JSON error via `internal/output` with a non-zero exit.
On success it prints nothing at all, so `run-shell` opens no output view.
`cli.go` registers the verb and adds `"switch"` to the `PersistentPreRunE` skip guard beside `"watchdog"`, so the verb never resolves a cwd, a location, config or geometry.
`cli.go`'s header comment names the verb.

Bound: the verb only ever runs `list-sessions`, `display-message` and `switch-client` on the told socket, against the told client.

Tests:
`switch_test.go` (untagged) is one table over `nextSessionID` (forward, backward, wrap at both ends, numeric order across `$9`/`$10`, single session, current missing) plus fake-`execHook` rows asserting the exact argv sequence and that one session issues no `switch-client`.
`switch_integration_test.go` (`//go:build tmux && linux`, run through the package's existing `tmuxkit.Main`) boots three sessions on a private socket with an attached client harness (as `attachgeometry_integration_test.go` attaches one), switches next and previous across the wrap, and asserts the client's session after each; an unknown client and a socket path with nothing listening return errors.
`internal/reedcli/cli_test.go` gains rows: neither flag, both flags and a missing `--client` each exit non-zero with a JSON error, and the verb is reachable with no git repository in the cwd.

Docs: `reedengine` `doc.go` adds `switch-client` to the subcommand set as an optional verb used only by this function; `docs/overview.md`'s reed line lists `switch` among the verbs; `SANDBOX-REED-SUITE.md` gains a mission that, with two worktrees' sessions up and a client attached, runs `lyx reed switch --next`/`--prev` with the socket path from `tmux -L <socket> display -p '#{socket_path}'` and checks the client lands on the neighbouring session (`PATTERN-sandbox-coverage`).

**ImpactSummary:** new verb and one engine-less function; `TmuxCmd` gains a socket-path mode, every existing caller unchanged.
