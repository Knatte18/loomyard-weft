---
verdict: APPROVED
---

The diff makes the change the board entry asks for.
Off Windows, `probeCapabilityLocked` runs `-V` and `list-commands` on a fresh private `-L` key with `-f /dev/null`, so the transient server `list-commands` starts never stands on the hub's socket.
Both callers (`lifecycle.go:361`, `window.go:59`) go through that one method, so no code path runs `list-commands` on the hub socket any more.
Windows keeps the engine's own `TmuxCmd`, unchanged.

Focus directive results:

- Race closure: held. A client of the hub socket cannot reach the probe's server, and resume goes through the same method as `up`.
- Cleanup and error path: held. The deferred `removeSocketFileOnceGone` runs after a failed probe too, returns at once when no socket file exists, logs its own failure and never replaces the probe's error.
- Socket path length: held. The hub key is `lyx-<base>-<hash8>` (`server.go:60`), at least 13 bytes, and the standalone key `lyx-<hash8>` is 12 bytes, equal to the probe key `lyx-p` plus seven hex digits.
- Summary claims (key length, 2s `socketGoneWait`, Windows unchanged, one extra tmux server per `up` or `resume`): each matches the diff.
- Docs and tests: `doc.go` matches the new behavior, and the new tmux-tier test pins the hub socket staying empty and no `lyx-p` socket left behind.
  No existing test covers that, and the non-parallel claim holds because a hub key never starts with `lyx-p`, since `p` is not a hex digit.

## Rejected

- Hermetic probe test does not assert the probe's `-L` key (code-tests, LOW): rejected, because the tmux-tier test is the lowest tier that can show the race and fails if the probe returns to the hub socket; an argv row would pin the implementation instead of the behavior (`PATTERN-test-economy`).
