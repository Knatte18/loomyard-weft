# Card 13 — the landing merge-in verify marks the driver strand

**Edit:**
- `internal/landingshed/deps.go`
- `internal/landingshed/verifygate.go`
- `internal/landingshed/verifygate_test.go`
- `internal/landingshed/doc.go`
- `internal/loomcli/landingdeps.go`
- `internal/loomcli/landingdeps_test.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/sharedbootstrap.go`

**Uses:**
- `internal/loomcli/bootstrap.go`

**Intent:**
Publish's and Finalize's merge-in verify show on screen on the run's driver strand, since no agent strand waits there (D2).

`landingshed.Deps` gains a told callback, `VerifyWaitMark func(label string, start time.Time) error`, where an empty label clears the mark; nil means "no strand to mark", following `CommitStatus`'s nil-is-absent convention, so a run with no driver strand (the Go driver) gets the status note only.
`verifyGate` carries it beside its other told values, and `check` clears the mark at the start of every verify, sets it (`verify <producer>`, started now) immediately before `g.verify`, and clears it on every path out of the verify: pass, skip, dirty, failure, timeout, error and cancel.
Every callback failure is logged at Warn and changes no Stuck reason, error return or verify outcome.
Bound: the mark is display only; the verify's result is decided exactly as before.

`loomcli` wires it to the driver strand it already resolves: `landingDeps` takes the callback as a new parameter, and `arm.go` and `sharedbootstrap.go` build it over loom's reed engine.
The callback reads reed's strand table at call time, picks the driver strand with `findDriverStrand`, and calls `SetWaitMark` on it only when that strand is live and not retiring; a strand that is gone, dead or replaced makes the call a logged Info no-op returning nil.
It is built as a small function over two seams (the strand read and the mark op) so `landingdeps_test.go` can pin, without tmux, the live-strand call, the gone-strand no-op and the mark-op error passed through for the gate to log; the every-field-populated drift guard covers the new field.

`verifygate_test.go` pins the clear-set-clear sequence around a passing verify, the clear on a failing, a dirty and an erroring verify, a nil callback, and a failing callback leaving the Stuck reason and error exactly as without it.
`doc.go` records the callback and its bound.

**ImpactSummary:** One optional dep and its wiring at loom's two landing-deps call sites; landing outcomes are unchanged.
