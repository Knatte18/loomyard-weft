# Card 23 — shuttleengine-run-literals

**Edit:**
- `shuttleengine/attach_test.go`
- `shuttleengine/attach_asking_test.go`
- `shuttleengine/gate_test.go`
- `shuttleengine/gate_list_test.go`
- `shuttleengine/gate_pending_test.go`
- `shuttleengine/run_test.go`
- `shuttleengine/run_inject_test.go`
- `shuttleengine/session_test.go`
- `shuttleengine/startup_test.go`
- `shuttleengine/wait_test.go`
- `shuttleengine/waiting_test.go`
- `shuttleengine/fixture_test.go`

**Intent:**
Finish #324 §4: the hand-built `&Run{runner:, spec:, runDir:, state:, clock:, deadline:}` literals over private fields go through `fx.newRun`, and the repeated `Config{...}` literals that match `fastConfig` or `defaultConfig` use those, so a `Run` field change touches the fixture alone.

A `Config` literal that differs from both shapes (a test whose subject is one timeout or interval) stays a literal, written as a modification of the nearer shape when that reads more clearly.
`rundir_test.go` and `spec_test.go` test `Config`/`Spec` values themselves and are left alone.
Extend `fixtureOpt`/`runOpt` only for variants several call sites share.

Protected Completion Signal regression tests (D9): restructure only, every row keeps its assertion.
Apply D10 to every touched test.

**ImpactSummary:** Untagged shuttleengine tests only; private-field Run literals and duplicate Config literals route through the fixture, assertions unchanged.
