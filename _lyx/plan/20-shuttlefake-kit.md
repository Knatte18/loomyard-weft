# Card 20 — shuttlefake-kit

**Create:**
- `testkit/shuttlefake/shuttlefake.go`
- `testkit/shuttlefake/shuttlefake_test.go`

**Uses:**
- `internal/shuttleengine#Engine`
- `internal/shuttleengine#ReedOps`
- `testkit/doc.go`

**Intent:**
Provide the one shared `shuttleengine.Engine` fake and `shuttleengine.ReedOps` fake (#324 §1), so adding an `Engine` or `ReedOps` method touches the kit and the in-package `shuttleengine` fake instead of every consumer.

`shuttlefake.go` imports only `internal/shuttleengine` and `internal/reedengine` and defines:

- `Engine`: inert by default, with one optional func field per method (`PrepareFn`, `ParseEventsFn`, `StartupFn`, `InterruptSequenceFn`, `TrustDismissSequenceFn`, `ComposeSendFn`, `ModelSwitchSequenceFn`, `AuditForksFn`, `AuditForksIncrementalFn`) and call recording the consumers read today (`PrepareCalls`, `LastPrompt`, the last `Spec`).
  The inert answers match what the surveyed copies return unscripted; where copies disagree (`Startup`'s default state), the default is the one most consumers use and the others set the func field.
  A smoke engine that launches a real shell embeds `Engine` and overrides `Prepare` only.
- `Reed`: a mutex-guarded strand table.
  `AddStrand` mints a GUID and marks it live, `RemoveStrand` retires it and records the GUID, `Status` reports the table; optional `StatusFn`/`StatusErr`, `RemoveErr` and per-method func overrides cover the scripted consumers; `SendText`, `SendKey` and `CapturePane` record their calls and answer from optional funcs.
- Compile-time assertions that `*Engine` and `*Reed` satisfy the two seams.

It asserts nothing (D3): no method calls `t.Errorf`, and the fakes take no `testing.TB` at all.
Package doc names the seam it fakes and the in-package exception: `internal/shuttleengine`'s own tests keep their richer fake because an import cycle forbids the kit there (D1).

`shuttlefake_test.go` covers the inert defaults, each override path, strand liveness across add/remove, and concurrent `AddStrand`/`Status` calls on one `Reed`, which `go test -race` must pass; the card's `Verify` runs it under the race detector, which the plain package run does not.

**Verify:** go test -race ./internal/testkit/shuttlefake/...
