# Reed hygiene: reachable Debug trace, refused socket path, hermetic servers, backed-off poll mode

Both batches recorded `done`.

## Batch 01 (cards 1 to 6), head `594297e9d`

- The durable logger sink now admits Debug records at Debug verbosity.
- The detached watchdog daemon's environment drops `LYX_LOG_FILE` and keeps `LYX_LOG_LEVEL`.
- `lyx reed watchdog` keeps its stderr half, and its help names the two Debug-trace routes.
- Each watched session's engine drives the daemon's told tmux binary.
- The resize watch loop's poll mode waits from a re-armed timer that backs off.
- `tickerPeriodFor` is deleted with its test step.
- Deviation: `pattern/PATTERN-daemon-wakeups.md` changed outside the cards' code targets.
- The fork also rewrote a stale paragraph in `internal/reedengine/doc.go` about the daemon's discarded stderr.

## Batch 07 (cards 7 to 11), head `d6edcfbdd`

- A socket path over the OS limit is refused before any tmux round trip, naming `TMUX_TMPDIR`.
- The test kit's tmux config text is the `ConfigText` export.
- `-f /dev/null` is on the server spawn and the capability probe; per-session shell pins and the test-isolation bound are replaced.
- A fresh boot installs the window-resized clear and signal entry for every session.
- The idle-CPU report gains an after-hygiene row and names the children.
- Deviation: `internal/reedengine/zoom.go` changed, because `Up`'s zoom bracket issues a tmux call before the boot refusals; it now runs the same host pre-flight (`preflightBootHost`).
- The fork reports that `LYX_LOG_LEVEL=debug` writes no Debug lines, because the CLI root hook's `SetVerbosity` resets the level.
  That contradicts the wording in `logger.go`, the `reedcli` help and the docs.
  No code was changed for it, as card 11 forbids; it is left for a follow-up.
- The benchmark's counts are consistent with, but do not confirm, the watcher-round-trips hypothesis.

Neither batch's policy warnings blocked recording.
