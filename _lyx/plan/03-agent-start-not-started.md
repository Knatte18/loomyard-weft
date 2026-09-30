# Card 3 — surface a never-ready agent start to the producer's error

**Edit:**
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/startup_test.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/engine_test.go`
- `internal/shedadapters/singlellm.go`
- `internal/shedadapters/singlellm_test.go`
- `internal/shedadapters/burler.go`
- `internal/shedadapters/burler_test.go`

**Uses:**
- `internal/shuttleengine#ErrNotStarted`

**Intent:**
Make the `agent-start` class (decision record D2) reachable: `shuttleengine.Runner.RunGated` deliberately folds `ErrNotStarted` into a `(result, nil)` return with a died or timeout outcome, so a producer today cannot tell a provider that never came up from an agent that died mid-run.

`shuttleengine.Result` (`run.go`) gains `NotStarted bool`: true only on `RunGated`'s own not-ready branch, where `errors.Is(err, ErrNotStarted)` holds, and false on every other return, including a died or timeout outcome reached in `Wait`.
`Run` inherits it through `RunGated`.
The Completion Signal Invariant is untouched: this sets a field on an already-classified result and adds no negative-verdict site.

`burlerengine.Result` (`engine.go`) passes `NotStarted` through 1:1, beside `RunDir` and `Gate`.

The producers that turn a died or timeout outcome into a returned error wrap `shuttleengine.ErrNotStarted` with `%w` when the result's `NotStarted` is true, keeping their existing message text in front of it:
- `SingleLLM`'s died/timeout arm (`singlellm.go`).
- `Burler`'s two-consecutive-died arm (`burler.go`), when the final attempt's result is `NotStarted`.
  The one-retry ladder is unchanged: a first not-started attempt still retries.
A producer that starts an agent through `StartGated` or `Start` already returns `ErrNotStarted` in its chain and needs no change.
`Bouncer` degrades rather than erroring on a not-done run and is out of scope.

Tests:
- `TestStartup_RunGated_NotReady` also asserts `result.NotStarted` is true, and the existing not-`ErrNotStarted` sub-case asserts it false.
- A `burlerengine` round over a fake shuttle returning `NotStarted: true` reports it on its `Result`.
- `SingleLLM` over a fake returning `OutcomeDied` with `NotStarted: true` returns an error for which `errors.Is(err, shuttleengine.ErrNotStarted)` holds; with `NotStarted: false` it does not.
- `Burler` whose two attempts both return `OutcomeDied` with `NotStarted: true` returns an error wrapping `ErrNotStarted`; two ordinary died attempts do not.

**ImpactSummary:** Adds one boolean to `shuttleengine.Result` and `burlerengine.Result` and a `%w` on two producer error arms; every existing error text and outcome is unchanged.

**Commit:** 3: shuttleengine: report a never-ready start on Result; producers wrap ErrNotStarted
