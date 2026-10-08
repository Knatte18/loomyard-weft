# Card 10 — held-wait-mark

**Edit:**
- `internal/shuttleengine/waitmark.go`
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/waitmark_test.go`
- `internal/shuttleengine/waiting_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`
- `internal/loomcli/reviewstatus_test.go`

**Intent:**
Show a held turn end as a wait (D7), and keep it from suppressing batten's quiet notice.

In `waitmark.go`, a third label `held` joins `gate <entry>` and `background shells`.
`waitState` gains the held wait's start; `showWait` ranks a running gate entry first, then the background-shell wait, then the held wait.
`Wait` starts the held wait at a held turn end (where it logs the hold and calls `notifyHeld`), stamped with the run clock's time, and stops it at the next tick that reads a new event, so the following turn start, turn end or ask clears it; any finalize and every return of `Wait` clear it through the existing `endWait`.
The mark goes through `showWait`, so the `wait.yaml` marker and `SetWaitMark` pane mark are set and cleared together and stay display and status only (`PATTERN-wait-mark-display-only`).
`lyx loom status` already prints any live marker's label, so it shows `held` with no code change.

Batten must not count a held wait (D7's narrowing):
`shuttleengine` gains `ReadWaitMarkers`, every live marker under the run-directory root, which `ReadWaitMarker` now returns the first of, unchanged in its answer; and `WaitMarker` gains a `Held` method reporting the `held` label.
`battencli`'s `readAgentActivity` reports `waitLive` only for a live marker that is not held, so a held run's quiet notice still fires, while a gate or background-shell wait in any run still suppresses it.
This narrows the one reader that branches on a wait marker and widens nothing.

Tests:
- `waiting_test.go`: a held turn end sets the `held` mark with the hold's time and writes `wait.yaml` with kind `held`; the next event clears both; a Wait that returns while held leaves neither (a row beside `TestShellWaitMark`'s).
- `waitmark_test.go`: `ReadWaitMarkers` returns every live marker and skips dead and torn ones; `Held` is true only for the `held` label.
- `wire_seams_test.go`: a live `held` marker reports no wait, and a `held` marker beside a live `background shells` marker in another run directory reports a wait.
- `reviewstatus_test.go`: `TestShuttleWaiting` gains a row whose marker kind is `held` and whose note names it.

**ImpactSummary:** Held turn ends now write wait.yaml and a pane mark labelled held; batten's waitLive ignores held marks; loom status shows the label; no shuttle decision reads the mark.
