# Show a failed Slack toggle click on reed's status bar

A click on reed's `slack` toggle could fail before the toggle ran and leave no sign.
The click line sends the verb's output to `/dev/null`, and `lym slack toggle` set its bar seam (`showToggle`) only in `wireProduction`.
A pre-run refusal, such as a failed worktree resolve or failed wiring, aborted the verb before the seam existed, so nothing redrew the bar and nothing was logged.
The same command run by hand worked, which is why the failure looked silent.

Closes #212

## What changes

- `productionCLI` sets `showToggle` to `reedengine.ShowToggleOutcome` up front, so a toggle whose wiring failed still reaches the bar.
- `preRun` reports a failed `toggle` refusal on the clicked bar before it aborts.
- The toggle's own failure and a pre-run refusal share one helper, `reportToggle`.
  It reads the `--socket`, `--tmux` and `--client` flags from the command, redraws the bar with `lym: <error>` first, and logs the error at Warn.
- `ShowToggleOutcome` shows its message for ten seconds (`display-message -d 10000`).
  tmux's default display time is under a second, too short to read a refusal.
- `docs/overview.md`, the `slackcli` package doc and the verb's help text describe the new behaviour.

## Design decisions

- The trace log exists only in a lym worktree.
  A click from a directory that is no worktree is therefore shown on the bar alone, and the docs say so.
- The ten-second display time lives in `ShowToggleOutcome`, so every caller gets it, not only the Slack toggle.
  `internal/orchcli/remotecontrol.go` calls it for the remote-control button, whose messages now stay ten seconds too.
- The three flags are named by constants, and `reportToggle` reads them back from the command rather than through captured variables, because `preRun` runs before `RunE` and has no access to those variables.

## Check by hand

- A real tmux click on the `slack` toggle was not run.
  Click it from a session whose directory is no worktree and confirm the refusal shows on the bar for about ten seconds.
- Click the remote-control button and confirm its message stays for the longer display time and reads as intended.

## Merge danger

**Door:** two-way
**Blast radius:** reed
