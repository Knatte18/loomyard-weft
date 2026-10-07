# Card 14 — clock-seam

**Create:**
- `internal/shuttleengine#Runner.SetClock`

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/fixture_test.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/startup_test.go`

**Intent:**
D10, the seam half: a test outside `shuttleengine` can drive `Wait`'s time, which the replay corpus (card 15) needs.

The unexported `clock` interface in `wait.go` becomes the exported `Clock`, with its uses renamed in `run.go` (the `Runner` and `Run` fields) and in the package's own tests that name the type (`fixture_test.go`, `wait_test.go`, `startup_test.go`); `realClock` and the fakes keep their names.
The rename is an AST-aware identifier rename of the type `clock` (the fields keep the name `clock`, which is also a common word), followed by a grep confirming no use of the old type name remains; it is typed `Edit` because a symbol-level `Rename` group on `clock` would overlap the `wait.go` targets of cards 2, 5 and 6, which the plan gate refuses.
`Runner` gains `SetClock(c Clock)`, documented as the time seam for a caller that replays a run.

What this lets through, and its bound: a caller of `SetClock` replaces the run's time source, so it can skip real sleeping and make the deadline whatever its clock says.
`NewRunner` and `NewDetachedRunner` default to the real clock, no production wiring calls `SetClock`, and a `Run` takes the clock when `Start` or `Attach` builds it, so a later call never changes a run in flight.
A nil clock is a caller bug and is not guarded.

Coverage: `fixture_test.go`'s `withClock` applies the clock through `SetClock` instead of assigning the field, so every test built through `withClock` exercises the setter, and no new test is added; tests that assign `runner.clock` directly stay as they are.

**ImpactSummary:** Renames the unexported clock interface to the exported Clock across wait.go, run.go and three in-package test files, and adds Runner.SetClock, which no production wiring calls.
