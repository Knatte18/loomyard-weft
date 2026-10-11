# Card 2 — A start on a lone-`%` Claude box takes its startup sends

**Create:**
- `internal/shuttleengine/claudeengine/start_test.go`
- `plan:internal/shuttleengine/claudeengine#TestStart_LonePercentBoxTakesTheStartupSends` -> `func TestStart_LonePercentBoxTakesTheStartupSends(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/testkit/shuttlefake#Reed`

**Intent:**
Pins issue #549's failure end to end: a start whose pane shows a ready Claude box holding only `%` passes the color send and the skill-load send and returns a run.
Card 1's table rows pin the classifier; this test pins that shuttle's pre-write guard, the startup sends and the send verifier all read the box through it.

`start_test.go` is an external test file, `package claudeengine_test`, like `corpus_test.go`.
It drives `shuttleengine.NewRunner(...).Start` on a fake clock with:
- an engine type local to the test that embeds `*claudeengine.Claude` from `claudeengine.New()`, so `Startup`, `IdleSession`, `InputBoxText`, `ParseEvents` and the session-signal parser are Claude's own, and overrides only `Prepare`, which writes no settings and returns a `shuttleengine.Launch` with a non-empty `PromptLine`;
- a `shuttlefake.Reed` whose `AddStrandFn` returns a strand with a segment color, whose `StatusFn` reports that strand live with a pane id, and whose `CapturePaneFn` returns a ready Claude pane: transcript lines, then the input box drawn as `claudeengine.IdleInputBoxFixture` draws it, except that the caret line reads `❯ %`;
- the reed's paste hook appends the events lines of `testdata/corpus/skill-load-turn-end` to the run's events file when the skill-load message is pasted, so the load turn ends; a load turn Claude cannot verify is confirmed unverified and fails nothing, so the case's transcript need not be readable.

The spec names one skill and leaves `ColorByCaller` false.
The test asserts that `Start` returns a non-nil run and a nil error, and that the pasted texts include the color command and the skill-load message, in that order.
Before card 1 the color send fails as busy, logged at Warn, and the skill-load send fails with `ErrNotStarted`, so the test would fail.

The package sits at its row in `cmd/lyx/testdata/test-budget.yaml`; raise `internal/shuttleengine/claudeengine` by one for the new top-level test.

**ImpactSummary:** test-only: one new top-level test in `claudeengine` and its budget row raised by one.

**Commit:** 2: Pin a start on a lone-% Claude box through its startup sends
