---
round: 1
focus:
  - "Card 5 poll-mode backoff in the resize watch loop: does the re-armed timer double to a minute, reset when something changes, and stay within `PATTERN-daemon-wakeups`; does card 6 leave no caller of `tickerPeriodFor`?"
  - "Cards 7 and 9 boot refusals in `ensureServerAndSessionLocked`: are they placed after `validateBootConfig` and before `probeCapabilityLocked`, and does `TestUp_BootValidation`'s zero-tmux-calls assertion cover them as rows?"
  - "Card 7 `resolvedSocketDir`: is the socket path measured after symlink resolution, with `TMUX_TMPDIR` named in the refusal, and is the GOOS passed as a parameter per the `goos-as-parameter` decision?"
  - "Card 9 hermetic servers: does every server spawn and both capability-probe invocations carry `-f /dev/null`, while every other engine call keeps its argv, and do strand panes stop starting login shells?"
  - "Card 10 `pinBootOptionsLocked`: does the boot path install the window-resized clear and signal entry for every session exactly once, and is it reachable by a fake tmux?"
  - "Cards 1 to 4 Debug trace: does the logger admit Debug at Debug verbosity, does the spawned daemon env drop `LYX_LOG_FILE` and keep `LYX_LOG_LEVEL`, and do the per-session engines drive the daemon's told tmux binary?"
  - "Test economy across the diff: new tests that duplicate existing coverage, and behavior added with no test; check the `internal/reedengine` budget row raises against `cmd/lyx/testdata/test-budget.yaml`."
---

Focus is set from the plan's card index and shared decisions; no round has been judged yet.
