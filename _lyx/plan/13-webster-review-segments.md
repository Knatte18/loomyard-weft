# Card 13 — Webster and review specs name their segment

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/engine_test.go`
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncer_seed_test.go`
- `internal/treadleengine/judge.go`
- `internal/treadleengine/targeting.go`
- `internal/treadleengine/judge_test.go`

**Uses:**
- `internal/segmentcolor/segmentcolor.go`
- `internal/shuttleengine/spec.go`

**Intent:**
The remaining spawning modules name their role's segment on their launch specs, completing D13's rule that every spawning module's launch spec names one.

Segments, per D13's table:

- Webster's Merriam spec (`runlevel.go`, role `MerriamStrandRole`) and its recovery spec (`recoverbatch.go`): `segmentcolor.Webster`.
- Burler's reviewer and fixer specs (`burlerengine/engine.go`): `segmentcolor.Review`.
- Bouncer's judge and seed specs, at all four `shuttleengine.Spec` literals in `bouncer.go`: `segmentcolor.Review`.
- Treadle's judge and targeting specs (`judge.go`, `targeting.go`): `segmentcolor.Review`, since both are judges of a review loop.

Where a role constant is declared, its comment block gains the segment beside it, so a new spawn site in that module finds both.

Tests: each package's existing spec test asserts the built spec's `Segment` beside the role it already asserts (`runlevel_test.go`, `recoverbatch_test.go`, `engine_test.go`, `bouncer_judge_test.go`, `bouncer_seed_test.go`, `judge_test.go` for both treadle framings); no new test function.

**ImpactSummary:** nine spec literals in five packages gain one field each; their runs get a color, nothing else changes.
