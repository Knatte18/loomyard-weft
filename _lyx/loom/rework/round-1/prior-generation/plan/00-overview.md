---
format: 5
approved: true
language: go
---

# Plan: reed hygiene — a reachable Debug trace, a refused over-long socket path, hermetic servers and a backed-off poll mode

Three reed fixes land as one task, closing #495.
The watchdog daemon's Debug lines reach a log the operator can switch on (cards 1 to 3), its per-session watchers drive the told tmux binary so one shim counts every spawn (card 4), the resize watch loop's poll mode backs off under `PATTERN-daemon-wakeups` (card 5, with card 6 deleting the ticker period it leaves unused), an over-long socket path is refused at boot with `TMUX_TMPDIR` named (card 7), reed's own servers stop reading `~/.tmux.conf` and strand panes stop starting login shells (card 9, on the kit config export card 8 adds), the resize signal entry is installed at boot so a strand-less session's watcher leaves poll mode (card 10), and the idle-CPU benchmark is re-run with the children identified (card 11).
Every card runs on the decision record alone; the one carry-over review finding, the socket path measured before symlink resolution, is what card 7's `resolvedSocketDir` does.

## Card Index

1 — logger-debug-at-debug-verbosity — the durable trace sink admits Debug records when the process verbosity is Debug
2 — spawn-watchdog-env — the detached daemon's environment drops `LYX_LOG_FILE` and keeps `LYX_LOG_LEVEL`
3 — watchdog-verb-keeps-stderr — `lyx reed watchdog` stops discarding its stderr half and its help names the two Debug-trace routes
4 — daemon-engines-use-told-tmux — each watched session's engine drives the daemon's told tmux binary
5 — poll-mode-backoff — the resize watch loop's poll mode waits from a re-armed timer that doubles to a minute while nothing changes
6 — drop-ticker-period — `tickerPeriodFor`, left without a caller by card 5, is deleted with its test step
7 — socket-path-refused-at-boot — a socket path over the OS limit is refused before any tmux round trip, naming `TMUX_TMPDIR`
8 — kit-config-text — the test kit's tmux config text becomes the `ConfigText` export
9 — hermetic-reed-servers — `-f /dev/null` on the server spawn and the probe, per-session shell pins, and the test-isolation bound replaced
10 — resize-signal-entry-at-boot — a fresh boot installs the window-resized clear and signal entry for every session
11 — benchmark-rerun — the daemon idle-CPU report gains an after-hygiene row and names the children

## Shared Decisions

### Decision: goos-as-parameter

A new pure function whose answer depends on the platform (`unixSocketPathLimit`, `serverSpawnArgv`) takes the GOOS as a parameter and its production caller passes `runtime.GOOS`, so one untagged table pins both branches; a method that only gates on the platform keeps reading `runtime.GOOS` directly, as `pinGeometryOptionsLocked` does today.

### Decision: boot-refusals-in-the-pre-tmux-block

Cards 7 and 9 add their refusals to `ensureServerAndSessionLocked` right after `validateBootConfig` and before `probeCapabilityLocked`, so `TestUp_BootValidation`'s zero-tmux-calls assertion covers them as rows; `validateBootConfig` keeps its signature, since `zoom.go` also calls it for a pure config check that must not resolve paths.

### Decision: boot-tail-extracted-once

Card 10 lifts the post-spawn pin block into `pinBootOptionsLocked` with the signature card 9 settled, so the boot path's re-sign happens once and the new hook install is reachable by a fake tmux; cards run 9 then 10 for that reason.

### Decision: probe-server-bound

The capability probe's `list-commands` may start a transient server on reed's socket ahead of `new-session`, and without `-f` that server reads the operator's config.
A config that creates a session keeps it alive, `sessionlessSocketHolderPersists` never reaps a server that lists a session, and reed's `new-session` would join it, against D7's rule that reed's own servers read no `~/.tmux.conf`.
So card 9 gives the probe's two invocations `-f /dev/null` as well, through a config-file field on the probe's copy of the engine's `TmuxCmd`, and every other engine call keeps its argv.

### Decision: test-budget-rows

`internal/reedengine` is at its budget row, so cards 2, 7, 9 and 10 each raise the row by the tests they add, to 243, 244, 246 and 247; `internal/reedcli` holds one test fewer than its row and card 4 fills it; `internal/logger` adds rows to an existing test only, and `internal/testkit/tmuxkit` adds no test.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
