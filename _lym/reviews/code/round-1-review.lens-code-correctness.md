```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff through the code-correctness lens.

**Per function and branch: the case I ran, and whether the result is correct**

- `loomengine.ClassifyHalt`, new `buildSkew` branch:
  - `(StateFailed, "stencilstore: …", false, true)` returns `(AnomalyBuildSkew, true)`. Correct.
  - `(StateFailed, "loop-exited…", true, true)`: the `interrupted` case comes first and returns `AnomalyInterrupted`. Correct, and matches the doc comment.
  - `(StateBlocked, "x", false, true)` returns `AnomalyEscalation`, so the flag is ignored. Correct, and `failedHalt` never sets the flag on a blocked halt anyway.
  - `(StateBlocked, ReasonBounceBudgetExhausted+"…", true, true)` returns budget-exhausted. Correct.
  - `(StatePaused, "", false, true)` hits the default and returns `("", false)`. Correct: the flag invents no kind for a non-failed state.
  - `(StateFailed, "", false, false)` returns `AnomalyProducerFailure`. Correct, unchanged.
  - The "at most one kind" claim holds because the `switch` returns on the first case that matches.
- `loomcli.failedHalt`, new `BuildSkew` field:
  - `err == nil` returns false before the `errors.As` call. Correct.
  - Real burler path: `stencilstore.Read` returns a bare `*NewerBoardCopyError` (`internal/stencilstore/reconcile.go:36`). `ReadSet` and `fillInstruction` pass it through unwrapped (`internal/burlerengine/prompt.go:97-99`). `composeReviewPrompt` and `Review` also pass it through (`internal/burlerengine/review.go:139-141`). `burlerreview.go:469` wraps it with `%w`, and `failureExit` returns it unchanged (`internal/shedadapters/burlerreview.go:398-403`). `shedengine` persists `callErr.Error()` and returns `callErr` (`internal/shedengine/run.go:406-409`), and `shedverbs` passes that same `err` to `AfterStep` (`internal/shedverbs/step.go:404-408`). So `st.Error == err.Error()` holds and `errors.As` finds the type. Correct.
  - Same path with a transient mark: `MarkTransient` wraps the error in `transientError`, which has `Unwrap` (`internal/shedengine/transient.go:41`) and the same `Error()` text, so `errors.As` still finds the type. Correct.
  - Persist failure: `errors.Join(callErr, persistErr)` makes the text differ from `st.Error`, so no note is written. That behaviour predates the diff and is unchanged. Correct.
  - Bouncer render path: `promptRenderError` has `Unwrap` (`internal/shedadapters/bouncer.go:1894`), so any failed halt it reaches keeps the chain. Its degrade arm becomes a `Stuck` (blocked) halt, which `failedHalt` never sees, so `BuildSkew` does not apply there. No wrong result.
  - The returned `haltNote` sets `BuildSkew` only on the `true` return path. Correct.
- `loomcli.writeHaltNote` passes `n.BuildSkew` through to `ClassifyHalt`. Blocked notes built in `loomAfterStep` and `loomPostRun` leave the field at its zero value (false). Correct.
- `loomPostRun` (`internal/loomcli/arm.go:580`) uses the same `failedHalt` on `runErr`, and `shedengine.Run` returns the `Step` error. Correct.

**Findings:** none.

**Tool calls**
- `grep` for `transientError` methods in `internal/shedengine/transient.go`: my context showed `MarkTransient` but not whether `transientError` implements `Unwrap`, which decides whether `errors.As` sees through the transient mark.
