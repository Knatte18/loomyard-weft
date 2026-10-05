# Card 13 — `cycle` clears, `distill` compacts, and both write the note first

**Create:**
- `internal/orchcli/distill.go`

**Edit:**
- `internal/orchengine/state.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/cycle.go`
- `internal/orchcli/cli.go`
- `internal/orchcli/doc.go`
- `README.md`
- `docs/overview.md`
- `internal/orchengine/state_test.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchcli/cli_test.go`
- `internal/orchcli/smoke_live_test.go`

**Uses:**
- `internal/orchengine#RenderNoteTemplateFile`
- `internal/orchengine#RequestCycle`
- `internal/orchengine#CycleRequested`

**Intent:**
D6: an operator verb picks the cycle's mode, and no cycle in either mode clears or compacts without a passed note gate.

Request marker: `RequestCycle` takes the mode (`CycleClear` or `CycleCompact`) and the request time and writes both into the marker as JSON; `CycleRequested` becomes a read that returns the recorded request and whether one is pending.
A marker that does not parse is treated as stale.
`ClearCycleRequest` is unchanged.

Watcher, in `tickIdle`:
- A pending request whose request time is at least `HandoffTimeout()` before now is removed with a log entry naming its mode and age, and nothing is acted on.
  This one rule covers a request whose idle probe never passes and a marker a dead watcher left, across watcher restarts, since the time is on disk.
- The cycle's mode is the request's mode for a requested trigger and `cfg.Mode()` for an automatic (hard or soft) trigger; `State` records it as `CycleMode`, and records the request time as `CycleRequestedAt` (zero for an automatic trigger).
- Both modes now enter the handoff-requested phase, with a fresh note path under `HandoffsDir` and the note template file rendered first (card 12); the `DEFER` reply of a soft trigger declines in both modes, as it does in clear mode today.
- The phase's existing gate (note file written, a turn end read after it, idle probe passing) then starts clearing in clear mode, and in compact mode starts compacting with the focus stencil.
  Both move `LastHandoff` to the note, so a later fresh `start` resumes from the newest note in either mode.
- The handoff-requested phase of a requested cycle times out at the earlier of `PhaseEnteredAt` and `CycleRequestedAt` plus `HandoffTimeout()`; it returns to idle with the abort reason `handoff timed out`, as today.
Compact mode no longer reaches `startCompacting` from `tickIdle` directly; the compact-mode hold of a hard trigger after a failed compaction stays.
Bound: nothing is cleared or compacted unless the note gate passed on that cycle, and every request ends, acted on or abandoned, within the handoff timeout of its request time.

CLI: `lyx orch cycle` requests a clear cycle and the new `lyx orch distill` (in `distill.go`, mounted in `cli.go`) requests a compact cycle, whatever `cycle_mode` says.
`distill` is a cobra verb with a `Short`, a `Long` and JSON errors, returning the same `requested` and `watcher_live` envelope as `cycle` (`PATTERN-cli-cobra`); the two share one request helper.
The helper writes the marker, then checks `WatcherLive`, and when no watcher is live removes the marker again and reports `watcher_live: false`, so no later watcher acts on a request made while none ran.

Tests (fake `Session`, injected clock, shortened timeouts): `cycle` in compact `cycle_mode` clears and `distill` in clear `cycle_mode` compacts; an automatic trigger follows `cycle_mode`; compact mode writes the note, honours `DEFER` and moves `LastHandoff`; a request whose note is never written, or whose idle probe never passes, is abandoned within the timeout and clears or compacts nothing; a marker older than the timeout found at a watcher's first tick is removed and acted on by none; a restarted watcher measures the timeout from the request time; both verbs report `watcher_live` and leave no marker when it is false.
`watcher_compact_test.go`'s assertions that compact mode writes no handoff are reworded behavior and become the note-first assertions.
`TestSmokeOrch_CompactCycle` in `smoke_live_test.go` expects a note under `HandoffsDir` after the compaction.
`orchengine/doc.go`'s cycle-mode and compact-cycle sections, `orchcli/doc.go`'s verb list, `README.md`'s orch bullet and `docs/overview.md`'s orch row (`lyx orch start|status|cycle|distill|stop`) describe the two verbs and the note in both modes.

**ImpactSummary:** the orch request marker, the watcher's idle and handoff-requested phases and two CLI verbs; `cycle` changes meaning from "cycle in `cycle_mode`" to "clear", and compact mode gains the note step.

**Commit:** 13: orch: cycle clears, distill compacts, and both write the note first
