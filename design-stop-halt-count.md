# The driver gives each stop an identity, counts halts at a row with lym's retries, and records a halt's four facts

`stop-halt-count` · Task · feature, shed, loom · draft

Decision record: [Sketch](decision-stop-halt-count.md)

Depends on: [`stop-settlement-table`](design-stop-settlement-table.md), [`loom-verbs-await-park`](design-loom-verbs-await-park.md)

Filed from the Sketch `self-healing-runs`.

Takes: D3 whole and D7's recording steps; `stopclass.Settle` and the class constants from stop-settlement-table; `dobbyengine.AutonomyLevel` through `hubgeom.Autonomy` from autonomy-unattended-level, converted with `stopclass.Level(level)` in the driver wiring in `internal/loomcli/drivewiring.go`. It answers the Discussion-Review round-3 BLOCKING finding failed-state-halt-not-counted-or-recorded through the stop identity, as both added decisions rule.
Delivers: D3: the stop record written through `internal/state` beside the settle records, holding the open stop's key, its halt entry's history index and the four facts; the identity opened, closed and taken over across a driver restart as D3 says; settle records carrying the stop identity, the class and the halt entry's index; one function that computes both the halt count at a row and lym's settles of a class at the row over the whole history, counting and excluding exactly what D3 lists, so `wakeRestartGap` never restores a bound. D7's recording steps: every hand-back records `class:`, `settle:`, `halts_at_row:` and `halt:` on its history entry, or on the stop record when the stop left none, and carries them on the notice beside `autonomy:`; every hand-back to the orch reads `settle: orch` or `settle: operator`. D7's criterion, which needs the dobby's child read, is carried by severe-halt-slack-post.
Exports: Fixed for consumers: `shedengine.HistoryEntry` fields `Class`, `Settle`, `HaltsAtRow` and `Halt`; `shedrun.SettleRecord` fields `StopID`, `Class` and `HistoryIndex`; the stop record type and its path helper in `internal/shedrun`; the halt key, the row's producer and the stop identity joined, which readers treat as an opaque string; the count function in `internal/sheddriver`.

The part changes these packages:

- internal/sheddriver (about 820 changed lines)
- internal/shedrun (about 80 changed lines)
- internal/shedengine (about 90 changed lines)
- internal/loomcli (about 40 changed lines)
