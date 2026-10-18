# Support log — tmux-conf-concurrent-boot-flake

## Interview

A darn run holds no interview.

## Rejected alternatives

- Retry the probe's `list-commands` on `server exited unexpectedly`: it hides the probe's own failure but leaves the transient server on the hub socket, where a concurrent boot's `new-session`, `has-session` or `display-message` can still reach it while it exits.
- A hub-wide boot lock serializing every worktree's boot: a wider change to reed's locking than the race needs.
- A probe socket under `lyxtemp`'s run directory through `-S`: the run directory's path is longer than tmux's per-user socket directory, which pushes the socket path toward the `sun_path` limit the hub key is already checked against.
- Isolating only the test, booting each worktree on its own hub: the production race between two worktrees of one hub would stay.

## Review rounds

_No rounds yet._

## Question ledger

- Is the flake in the test or in production? Self-pick: production; two `lyx reed up` calls from two worktrees of one hub race the same way.
- Does the probe need `-f /dev/null` on a private socket? Self-pick: yes; an operator config that creates a session would keep the private transient server alive and leak it.
