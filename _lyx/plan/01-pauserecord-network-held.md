# Card 1 — pauserecord-network-held

**Edit:**
- `internal/pauserecord#Network` -> `func Network(op string) (held time.Duration, done func())`
- `internal/pauserecord#Gate`
- `internal/pauserecord#TestNetwork`
- `internal/pauserecord/doc.go`
- `internal/githubclient#authRT.RoundTrip`
- `internal/gitexec#runCore`
- `internal/pausecli#TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub`
- `internal/battencli#TestWire_PointsTheNetworkGateAtItsSlug`

**Intent:**
`pauserecord.Network` reports how long its pause hold kept the call waiting, so the GitHub transport can move the call's deadline later by that time (D6, issue #606).
It keeps its context-free signature: it still takes only the operation name and never a context.
It now returns `held` beside the `done` function it returns today.
`held` is zero unless the wait slept at least once; only then is it the time from `Network`'s entry until the wait returned, measured on the gate's clock.
`WaitUncovered` returns only an error, so `Network` tells whether it slept by wrapping the sleep it hands the `Waiter` (the gate's `Sleep`, or the timer sleep when that is nil) in a closure that notes it ran.
With no gate installed, and for a call no live entry covers, the wait never sleeps and `held` is exactly zero under any clock, so such a call keeps its deadlines unchanged; the call still writes its marker after the wait as today.

`Gate` gains a `Now func() time.Time` field, the clock `held` is measured on; nil means `time.Now`.
It lets a test advance the clock in the gate's `Sleep` seam, so `held` is pinned without wall time.

Every caller is adapted to the two results with no change of behavior:
`authRT.RoundTrip` and `gitexec.runCore` take `_, done := pauserecord.Network(...)` and `defer done()`.
Card 2 is where the transport starts using `held`.
`gitexec` already holds a remote command before its deadline starts, so it ignores `held` (D6).
The test callers `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` and `TestWire_PointsTheNetworkGateAtItsSlug` call `Network` as a statement or with `()` appended, so they are rewritten to bind both results and call `done`.

`TestNetwork` gains one row: a gate whose `Sleep` advances a fake `Now` by 3 s while a live entry covers the process returns `held` of exactly 3 s.
The existing rows keep their assertions with the two-result form.
The row "no gate is a no-op writing no marker" also asserts `held` zero.
The SetGateSlug row's gate gets a fake `Now` that advances by 1 s on every call, not only inside `Sleep`, and its first call, which no entry covers, asserts `held` zero, so the row fails an implementation that measures elapsed time unconditionally.

`doc.go` states that Network reports the time it held the call, and that it keeps its context-free signature.
The package keeps its import set: `time` is already imported.

**ImpactSummary:** One exported re-sign in a leaf package: two production callers (githubclient transport, gitexec runCore) and three test callers, all adapted mechanically; no behavior change outside the new `held` result.

**Commit:** 1: pauserecord: Network reports how long it held the call
