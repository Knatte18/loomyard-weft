---
round: 1
focus:
  - "internal/reedcli/watchdog.go `runWatchdogLoop`: when `unwatched` is set, does consuming the discover signal on the timer wake preserve the stop-wake semantics, the backoff and the discovery ceiling, and can a stop marker be consumed twice or lost between the timer read and `discoveryWake.onSignal`?"
  - "`TestWatchdogDaemon`'s shared discovery loop now uses a never-delivering opener: does it still guard the stop a `down` writes before the next `up`, and does the new failing-opener stop row in the exit-path table actually exercise the fallback rather than the real watch?"
  - "internal/reedengine tmux floor cache (`tmuxFloorAnswer`) and `launchStrandLocked`: is the version parse and the below-3.5 comparison in `toolfloor.TmuxHasAllowSetTitle` correct for edge lines (e.g. `tmux 3.5a`, `tmux next-3.6`, `tmux 3.10`), and does an unparseable line, Windows or a hook-driven test command still try the option and Warn on failure?"
  - "Is the fallback Info logged once per process as the board entry asks, given the cache is per resolved tmux path, and is the cache access race-free under concurrent strand launches?"
  - "Scope against the board entry: `tmuxkit.ServerEnviron` keeps its `LYX_TRACE_ID` literal against the decision record; check the import-cycle claim and that the remaining three test files all name `logger.TraceIDEnv`, with no other literal left behind in those files."
  - "Repository rules: `doc.go` updates for `toolfloor`, `reedengine` and `reedcli` land in the diff, `PATTERN.md` needs no new invariant, and the summary's claims match the diff."
  - "Test economy: new tests for `TmuxHasAllowSetTitle`, the skipped `set-option` and the unwatched stop row; flag a duplicate only by naming the existing covering test, and flag any added behavior with no coverage."
---

Seed focus drawn from the board entry's three fixes and the change description's claims.
