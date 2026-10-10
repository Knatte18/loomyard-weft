# Fixer report — discussion, round 2

All six findings were checked against the code and fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` needed no change.

## Fixed

- **F1 (MEDIUM, design):** D6 now resolves `$TMUX_TMPDIR`, or `/tmp`, through symlinks before appending `tmux-<uid>` and the key, and keeps the unresolved path when resolution fails, as tmux does.
  It names the symlinked `TMUX_TMPDIR` and macOS `/private/tmp` cases and states that `socketDirFromEnv` still holds the directory rule while the resolution belongs to the boot check alone.
  The acceptance criterion now pins the pure function on the resolved directory and adds a row for a short symlink that points at a long directory.
- **F2 (LOW, design):** I re-checked this live: under tmux 3.6, `list-sessions` past the limit printed `File name too long` and exited 1.
  The D6 rationale no longer says tmux exits 0.
  It now says the boot loop starts the server without waiting on it and reads each `has-session` connect failure as a server that is not up yet.
- **F3 (LOW, decision):** D5 now sets the cadence for each of the other `handleWatchOutcome` outcomes.
  A deferral returns it to the base, a failed re-apply doubles it, and a degraded-box query doubles it unless that tick applied a layout.
  The poll-mode acceptance criterion now lists those rows.
- **F4 (LOW, consistency):** The D5 bound now ties the lifetime case to a failed file watch alone, which matches `promotionBlocked`.
  A failed hook install now shares the older-lyx wording: the watcher stays in poll mode until a later attach or apply installs the hook, because poll mode keeps probing.
- **F5 (LOW, scope):** The D7 bound now says the skip applies from the hub server's next start.
  A server an older lyx started keeps the operator's config until it exits, and its new sessions get only the shell pins.
- **F6 (LOW, consistency):** The second Debug route in D2 is now: while no daemon runs for the hub, run `lyx reed up`, `attach` or `resume` with `LYX_LOG_LEVEL=debug` set.
  The help also states that a daemon already running keeps its starting level, because a new spawn finds the hub lock held and exits.

## Disputed

None.

## Deferred

None.
