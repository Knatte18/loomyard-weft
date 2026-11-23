# Study: a lym-started Landlock area instead of bwrap

`landlock-area-study` · Task · research, shuttle, reed, claude · running · Drift-Verify

## Aim

Find whether lym can close the two exposures in note `sandbox-closed-area-build` without `bwrap`, root or AppArmor, by applying Landlock to its own process before it starts reed's tmux server and the agents, so every child inherits the restriction.
The two exposures: an agent reaching `/run/user/<uid>/bus` (and so `systemd-run --user`), and an agent typing into other hubs' tmux panes through the sockets in `/tmp/tmux-<uid>`.

## Questions

- Which Landlock ABI each machine's running kernel supports: Neve (this machine) measured; OSL-1046 from its kernel version as recorded in the repo or docs, marked unmeasured.
- Whether that ABI can deny connecting to a pathname unix socket (the D-Bus socket, other hubs' tmux sockets) while allowing the hub's own tmux and creel sockets, and what the filesystem rules must allow for git, Go, Claude Code and lym to work.
- Where in lym the restriction is applied (which process, before which spawn), and what breaks: `lym reed attach` from outside, landing deploy, `lym reed window`, `lym ide`, hibernate.
- What Claude Code PreToolUse hooks add as a second line, and what they cannot stop (a script or test binary run by the agent).

## Bounds

One small Go probe program that applies a Landlock ruleset and tries the socket connects and file writes, run on Neve only, at most three runs.
No change to lym's code.
The report goes to `docs/research/platform/landlock-area-2026-11-23.md`, ending with a recommendation and the tasks it would take.
