# The settlement table: stop classes and who settles each, per autonomy level and halt count

`stop-settlement-table` · Task · feature, shed · draft

Decision record: [Sketch](decision-stop-settlement-table.md)

Filed from the Sketch `self-healing-runs`.

Takes: D2's table and its verdict order; nothing from another part.
Delivers: D2's table steps: the new leaf package `internal/stopclass`, standard library only and no I/O, holding one named class constant per row of D2's table and the one function that returns a verdict for a class, a level and the halt count at the row, read in D2's order. lym's bound per class lives here, and lym's column applies at every level. The driver rows that take their bound from it, and the `settle: orch` hand-back, are built by `driver-settles-stops`.
Exports: Fixed for consumers: `stopclass.Class` with one constant per row; `stopclass.Level` with `LevelOff` ("false"), `LevelOn` ("true") and `LevelUnattended` ("unattended"), the same strings `dobbyengine.AutonomyLevel` holds, so a consumer converts with `stopclass.Level(level)`; `stopclass.Verdict` with `VerdictLym`, `VerdictOrch`, `VerdictOperator`; `stopclass.Settle(class Class, level Level, haltsAtRow, lymSettles int) Decision`, `Decision{Verdict, Bound}`.

The part changes these packages:

- internal/stopclass (about 480 changed lines)
