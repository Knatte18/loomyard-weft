# Run the PR-Review session tests over real reed and a recorded tmux

The PR-Review session tests stood in for reed with hand-written closures and a fake strand table.
They now run the real `shuttleengine.Runner` over the real `reedengine.Engine`, with no tmux needed.
Below reed, tmux calls, pane ages and the systemd user manager's memory caps are replayed from recordings;
reed's other process-table reads still go to the host.
The provider engine (`shuttlefake`) is the one stand-in between the session runner and reed.

## What changes

- `reedengine.NewOver` builds an engine over a `reedengine.Outside`: tmux, pane age, memory cap and strand GUID source.
  A zero field selects the real thing, and `New` is `NewOver` with an empty `Outside`.
- A new kit, `internal/testkit/tmuxrec`, records the tmux, pane-age and memory-cap answers while reed drives a scenario over real tmux, and replays them.
  It mints the strand GUIDs itself.
  Each recording names its command, date, tmux version and OS.
  One test replays every scenario without tmux, and another drives them over real tmux to compare what reed read, or to rewrite the recordings with `-tmuxrec.write`.
- `sessionLiveness` and `sessionRegistrar` move from `loomcli` to `reedengine` as `SessionLiveness` and `SessionRegistrar`, so the tests use the same code as the production wiring.
  Their error messages now start with `reedengine:`.
- `resolveShellPath` is exported as `ResolveShellPath`, and `paneProcessAge` as `PaneProcessAge`, for the kit.

## Design decisions

- The recording seam is below reed, at tmux, so that reed itself is never stood in for, as `PATTERN-test-stand-ins` requires.
- A replay answers only recorded calls.
  Changes must come in their recorded order, and a read is never answered from before the latest change the replay passed.
- Paths, names and process ids are stored as placeholders.
  Replayed pids lie above the Linux pid range, so none names a host process to be signalled or probed.
- With a told tmux, the session spawn goes through it too.
  That path skips the spawn's own tmux version-floor check, and passes tmux only the `new-session` subcommand, without the server flags.
- The `MailAgent` closures in these tests and the stand-ins in `envkit` are unchanged and left for later.
  A real creel path needs a daemon.

## To check by hand

- Off Linux, `PlaySession` skips, because the recordings were made on Linux.
  Every PR-Review test built on the shared session fixture therefore skips on other systems until recordings are made there.
- Registration check in `internal/landingshed/prreview_session_test.go`: confirm the exact pane-pid comparison against `startedPanePID` is as strict as the strand check it replaces.
- Host dependence of a replay (`internal/testkit/tmuxrec/session.go`): the shell path is a placeholder and another OS skips.
  Confirm nothing else host-specific is left in the recordings.
- Call stdin (`internal/testkit/tmuxrec/record.go`, `tmuxrec.go`): it is neither recorded nor matched.
  No current recording holds a `load-buffer` call, the paste call that writes stdin.
  Each paste names a fresh random buffer in its arguments, so a replayed paste matches no recorded one and fails as unanswered.
  A scenario that pastes cannot be replayed as the kit stands.
- Replay refusals (`internal/testkit/tmuxrec/recordings_test.go`): confirm `TestReplay_RefusesWhatItsRecordingCannotAnswer` covers the refusals you rely on.
- Stale reads (`Replay.answer` in `internal/testkit/tmuxrec/tmuxrec.go`): confirm a read after a change cannot be answered with the state from before it.

## Merge danger

**Door:** two-way
**Blast radius:** `sessions`
