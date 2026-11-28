# The settlement table of stop classes, loom verbs that wait for the driver to park, and the driver's halt count

`stop-settlement-table` · Task · feature, enhancement, shed, loom, batten

Decision record: [Sketch](decision-stop-settlement-table.md)

## The settlement table: stop classes and who settles each, per autonomy level and halt count

Filed from the Sketch `self-healing-runs`.

Takes: D2's table and its verdict order; nothing from another part.
Delivers: D2's table steps: the new leaf package `internal/stopclass`, standard library only and no I/O, holding one named class constant per row of D2's table and the one function that returns a verdict for a class, a level and the halt count at the row, read in D2's order. lym's bound per class lives here, and lym's column applies at every level. The driver rows that take their bound from it, and the `settle: orch` hand-back, are built by `driver-settles-stops`.
Exports: Fixed for consumers: `stopclass.Class` with one constant per row; `stopclass.Level` with `LevelOff` ("false"), `LevelOn` ("true") and `LevelUnattended` ("unattended"), the same strings `dobbyengine.AutonomyLevel` holds, so a consumer converts with `stopclass.Level(level)`; `stopclass.Verdict` with `VerdictLym`, `VerdictOrch`, `VerdictOperator`; `stopclass.Settle(class Class, level Level, haltsAtRow, lymSettles int) Decision`, `Decision{Verdict, Bound}`.

The part changes these packages:

- internal/stopclass (about 480 changed lines)

## lym loom resume and lym loom start wait for the driver to park, with a no-wait option batten passes

Filed from the Sketch `self-healing-runs`.

Takes: D5 whole: the `shedrun.StartNotParkedKind` refusals in `internal/loomcli/resume.go` and `internal/loomcli/start.go`, the driver's wait record `sheddriver.ReadDriveWait` that push-retry-not-halt added, the keep-warm cap from `shuttleengines.KeepWarm`, `childSpawnError` in `internal/battencli/wire.go` and the `battenshed.ErrChildNotParked` retry in `internal/battenshed/innerrun.go`.
Delivers: Both verbs wait for the park, or for a run lock on a non-running run to be released, under the keep-warm cap, waking at most once a second with an injected clock; answer at once for a driver that keeps a wait record; refuse past the cap naming the retry; and refuse at once with the no-wait option, which batten passes on its child start and on every resume it runs, the resume of dobby-pause-lift-resume's lift pass among them.
Exports: Fixed for consumers: the `--no-wait` flag on `lym loom resume` and `lym loom start`.

The part changes these packages:

- internal/loomcli (about 420 changed lines)
- internal/battencli (about 60 changed lines)
- internal/battenshed (about 50 changed lines)

## The driver gives each stop an identity, counts halts at a row with lym's retries, and records a halt's four facts

Filed from the Sketch `self-healing-runs`.

Takes: D3 whole and D7's recording steps; `stopclass.Settle` and the class constants from stop-settlement-table; `dobbyengine.AutonomyLevel` through `hubgeom.Autonomy` from autonomy-unattended-level, converted with `stopclass.Level(level)` in the driver wiring in `internal/loomcli/drivewiring.go`. It answers the Discussion-Review round-3 BLOCKING finding failed-state-halt-not-counted-or-recorded through the stop identity, as both added decisions rule.
Delivers: D3: the stop record written through `internal/state` beside the settle records, holding the open stop's key, its halt entry's history index and the four facts; the identity opened, closed and taken over across a driver restart as D3 says; settle records carrying the stop identity, the class and the halt entry's index; one function that computes both the halt count at a row and lym's settles of a class at the row over the whole history, counting and excluding exactly what D3 lists, so `wakeRestartGap` never restores a bound. D7's recording steps: every hand-back records `class:`, `settle:`, `halts_at_row:` and `halt:` on its history entry, or on the stop record when the stop left none, and carries them on the notice beside `autonomy:`; every hand-back to the orch reads `settle: orch` or `settle: operator`. D7's criterion, which needs the dobby's child read, is carried by severe-halt-slack-post.
Exports: Fixed for consumers: `shedengine.HistoryEntry` fields `Class`, `Settle`, `HaltsAtRow` and `Halt`; `shedrun.SettleRecord` fields `StopID`, `Class` and `HistoryIndex`; the stop record type and its path helper in `internal/shedrun`; the halt key, the row's producer and the stop identity joined, which readers treat as an opaque string; the count function in `internal/sheddriver`.

The part changes these packages:

- internal/sheddriver (about 820 changed lines)
- internal/shedrun (about 80 changed lines)
- internal/shedengine (about 90 changed lines)
- internal/loomcli (about 40 changed lines)

