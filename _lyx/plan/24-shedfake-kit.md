# Card 24 — shedfake-kit

**Create:**
- `testkit/shedfake/shedfake.go`
- `testkit/shedfake/shedfake_test.go`

**Uses:**
- `internal/shedengine#ShedProducer`
- `testkit/doc.go`

**Intent:**
Provide one shared fake for each shed-layer seam and the producer `Call` helpers (#324 §2), so a `Shuttle` or `ShedProducer.Call` change touches one file instead of one per package.

`shedfake.go` imports only `internal/shuttleengine`, `internal/burlerengine`, `internal/shedengine` and `internal/websterengine`; it satisfies `shedadapters.Shuttle`, `shedadapters.BurlerRunner` and `mergeresolve.Shuttle` structurally, never importing those packages, so `shedadapters`' and `landingshed`'s in-package tests can use it.

- `Shuttle`: the superset of `shedadapters/singlellm_test.go`'s `fakeShuttle` (the richest copy, D1) with exported fields: scripted `Result`/`Err` and `AttachResult`/`AttachFound`/`AttachErr`; recorded `Called`, `GotSpec`, `Specs` (every spec, in order), `AttachCalled`, `GotAttachSpec`, `GotGateSpec`, `GotAttachGateSpec`; hooks `DuringRun`/`DuringAttach`; `GateAttempts` and `GateReason`; and an optional `RunFn` that replaces the scripted `Run` answer for a consumer that dispatches per spec (loomrecipe's sequence shuttle).
  `RunGated`/`AttachGated` follow the one "every test fake evaluates the gate once" contract, exactly as the singlellm copy does.
- `BurlerRunner`: scripted `Results`/`Errs` per invocation (the last entry repeats once exhausted), recorded `GotProfiles`, `GotOpts`, `Calls`, and a `DuringRun(callIndex)` hook.
- `MergeShuttle`: answers `Run` with a zero `Result` and nil error, with an optional `RunFn`.
- `WebsterSeams() websterengine.RunDeps`: a `RunDeps` whose `Starter`, `Reed`, `Engine` and `RefMatcher` are the embed-the-interface placeholders, non-nil for constructor checks and panicking if called.
- `CallOK(t testing.TB, p shedengine.ShedProducer) (shedengine.Outcome, shedengine.OutputPointer)`: calls `p.Call(context.Background())` and `t.Fatalf`s on a non-nil error.
- `RequireOutcome(t testing.TB, p shedengine.ShedProducer, want shedengine.Outcome) shedengine.OutputPointer`: `CallOK`, then `t.Errorf("Call() outcome = %q; want %q", ...)` on a mismatch, continuing as the inline boilerplate does.

`Fatalf`-on-error stays exact (D9): some tests rely on stopping there.
Those two helpers are the only assertions the kit makes (D3).
Package doc names the cycle-forced exceptions: `internal/burlerengine`'s in-package tests keep their own `Shuttle` fake, because the kit imports `burlerengine`.

`shedfake_test.go` covers the gate-once contract for both gated methods (pass, fail with reason, closure error, gate skipped on a non-Done outcome), `BurlerRunner`'s repeat-last scripting, `WebsterSeams` non-nil fields, and `CallOK`/`RequireOutcome` against a stub producer on their passing path.
