# Card 11 — shuttle marks its post-turn waits on disk and on screen

**Create:**
- `internal/shuttleengine/waitmark.go`
- `internal/shuttleengine/waitmark_test.go`

**Edit:**
- `internal/shuttleengine/reed.go`
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/gate_test.go`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/doc.go`
- `internal/testkit/shuttlefake/shuttlefake.go`

**Uses:**
- `internal/reedengine/io.go`

**Intent:**
Shuttle's two Go-side post-turn waits become visible: any gate entry it evaluates, and the background-shell wait (`background_shell_wait_min`) after a turn end (D1, D2).
This card only surfaces the background-shell wait; its policy, `expiredTurnEnd`'s bound and classification, is unchanged (`shuttle-turns` owns it).

On disk, `waitmark.go` declares the wait marker, a small YAML file in the run's own run directory (`<runDir>/wait.yaml`, under the `.lyx` shuttle root `runDirRoot` already builds, so no new `_lyx`/`.lyx` literal): its kind (`gate <entry name>` or `background shells`), its start time and the pid of the process running `Wait`.
It is written when the wait starts and removed when it ends.
The exported reader `ReadWaitMarker(cfg Config, anchorPath string) (WaitMarker, bool, error)` scans the run directories under `runDirRoot(cfg, anchorPath)` and returns the first marker whose pid is alive; a marker with a dead pid reads as absent and the scan goes on, the same rule `verifytree.ReadMarker` uses, through a package liveness seam over `proc.IsAlive` so the tests stay tier 1.
An unreadable or undecodable marker is skipped with a logged Warn, never an error, so one torn file cannot hide another run's wait.

On screen, `ReedOps` gains `SetWaitMark(guid, label string, start time.Time) error`, which `*reedengine.Engine` already satisfies (card 1); the run's mark goes on its own agent strand.

Where the waits start and end, in `wait.go`:

- a gate entry: `evaluateGate` writes the marker and sets the mark (`gate <entry name>`) immediately before calling an entry's closure, and removes both immediately after the closure returns, on every path out (pass, fail, pending, error), including the final evaluation `finalize` runs;
- the background-shell wait: the marker and mark (`background shells`) are on exactly while the recorded waiting turn end's outstanding tasks are all non-awaited background shells not yet waited out, so they start when `expiredTurnEnd` first holds such a turn and end when the waiting list clears (the shells expire into a turn end, or a later event replaces the waiting turn end);
- `Wait` clears the strand's pane mark and removes any marker file of its run on entry, which covers a run that starts and one that attaches, so a mark left by a crashed step is gone at the next touch, and removes the marker and clears the mark on every return.

Every marker write or removal failure and every `SetWaitMark` failure is logged at Warn and changes no verdict, error return, re-prompt or gate outcome: a shuttle wait never ends differently because a mark could not be set or cleared.
The marker and mark are provider-neutral and live in `shuttleengine`, never `claudeengine` (`PATTERN-shuttle-provider-seam`), and the helpers sit in `waitmark.go`, so the audited negative-verdict sites `completionsignal_enforcement_test.go` pins in `wait.go` are untouched.

Tests: the fake reed (`fakes_test.go`) and `shuttlefake.Reed` record `SetWaitMark` calls and can fail them.
`gate_test.go` pins the mark set then cleared around a passing, a failing and an erroring entry, the marker file absent after each, and a failing `SetWaitMark` leaving the verdict and re-prompt as they were.
`waiting_test.go` pins the background-shell mark and marker present while only non-awaited shells are outstanding, gone once they are waited out and gone when a new event replaces the waiting turn end, and the stale-mark clear on `Wait` entry.
The new `waitmark_test.go` pins `ReadWaitMarker`: no runs, a live marker, a dead-pid marker skipped in favour of a live one in another run directory, and a torn marker skipped.
`doc.go` records the marker, the mark and their bound: display and status only, never read by a shuttle decision.

**ImpactSummary:** The ReedOps seam gains one method that reed's engine and both fakes implement; gate and shell-wait verdicts are unchanged.
