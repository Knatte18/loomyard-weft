```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, find the test row that pins it.

**Per behaviour, the test that pins it**
- **A goto onto PR-Review on a receiver with no runner (`c.runner == nil`), over a record that names a strand, does not dereference a nil runner.** Pinned by the new row `PRReviewWithARecordedStrandStopsItThroughARunnerOfItsOwn`. A nil `*Runner` would panic in `StopStrand` at `r.toldErr` (`internal/shuttleengine/stop.go:151`), and the test would fail.
- **A stop failure through the hook returns the `could not stop the recorded PR-Review session` step error.** Pinned by the new row's `wantErr` substring.
- **A stop failure leaves the record whole.** Pinned by the new row's `reflect.DeepEqual` against `withStrand`. At unit level it is also pinned by `TestClearPRReviewOnGoto/"a stop error clears nothing and commits nothing"` (`internal/loomcli/prreviewgoto_test.go:65`).
- **The hook builds its own runner through `stepStopRunner` and stops through `Runner.StopStrand`.** Nothing pins this; see finding 1.
- **The existing rows now compare the whole record with `reflect.DeepEqual`.** This tightens them and widens nothing. `AnotherRowKeepsTheRecord` and `PRReviewClearsTheAutonomyAndTheAskMark` still fail on the regressions they caught before.
- **The `Fatalf`→`Errorf` change on the hook's error.** This changes no behaviour, and the record read still runs after it.

**Findings**

1. MEDIUM, design. Location: `internal/loomcli/gotohook_integration_test.go:54`.
   - **Summary:** The new row's `wantErr` is the `prReviewClearFailure` prefix, which both failure branches of the stop closure produce. So the row also passes when `stepStopRunner` fails to build a runner, for example from a config or geometry load error. It does not pin the "runner of its own" stop that its name and doc comment state (lines 20–22).
   - **Evidence:**
     - `internal/loomcli/prreviewgoto.go`, `prReviewGotoAdmitted`: the closure returns `"load the shuttle and reed config to stop the session: %w"` when `stepStopRunner` fails, and `runner.StopStrand(guid)` otherwise. `clearPRReviewOnGoto` wraps both as `prReviewClearFailure("stop the recorded PR-Review session", err)`.
     - `internal/shuttleengine/stop.go:229`: the runner's own stop fails with `"shuttle: stop strand %s: probe reed before removing it: %w"` when no reed session is up.
   - **Fix:** Assert a substring that only the runner's stop yields, such as `"shuttle: stop strand guid-1: probe reed before removing it"`. Alternatively, also assert that the error lacks `"load the shuttle and reed config"`.
   - The row's name says the hook "StopsIt", but the row pins a failed stop. Rename it to match, for example `PRReviewWithARecordedStrandOnAnUnwiredReceiverReturnsTheStopError`.

**Tool calls**
- Read `internal/loomcli/prreviewgoto_test.go` lines 1–120, to check whether `clearPRReviewOnGoto`'s unit table already pins the record kept on a stop error. The manifest had only located the file, not read it.
