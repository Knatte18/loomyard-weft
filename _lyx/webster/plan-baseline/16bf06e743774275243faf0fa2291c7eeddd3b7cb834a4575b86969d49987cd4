# Card 4 — shuttle's reed seam carries the paste-submit and the activity query

**Edit:**
- `internal/shuttleengine#ReedOps`
- `internal/shuttleengine#fakeReed`
- `internal/testkit/shuttlefake#Reed`
- `internal/testkit/shuttlefake#TestReed_PaneTransportRecordsCalls`
- `internal/testkit/shuttlefake#TestReed_Overrides`

**Create:**
- `internal/shuttleengine#fakeReed.PasteSubmit`
- `internal/shuttleengine#fakeReed.OperatorActive`
- `internal/testkit/shuttlefake#PasteSubmitCall`
- `internal/testkit/shuttlefake#Reed.PasteSubmit`
- `internal/testkit/shuttlefake#Reed.OperatorActive`

**Uses:**
- `internal/reedengine#Engine.PasteSubmit`
- `internal/reedengine#Engine.OperatorActive`

**Intent:**
Widen the seam through which shuttle drives reed, so later cards can paste and ask for operator activity, and give both fakes the two ops (`PATTERN-testkit`: the fakes live in the existing kits).

`ReedOps` gains two interface methods, `PasteSubmit(guid, text string, submit bool) error` and `OperatorActive(guid string) (active bool, quiet time.Duration, err error)`, with its doc comment naming them as the pane-write and operator-activity transport ops; `*reedengine.Engine` already satisfies both after cards 1 and 2, and the compile-time assertion stays.

`shuttlefake.Reed` gains `PasteSubmitCalls []PasteSubmitCall` (`GUID`, `Text`, `Submit`), `PasteSubmitFn`, `ActivityCalls []string`, `OperatorActiveFn`, and `Quiet time.Duration`.
`PasteSubmit` records the call under the mutex and answers `PasteSubmitFn`, else nil; `OperatorActive` records the guid and answers `OperatorActiveFn`, else not active with `Quiet`, defaulting to 5 s.

shuttleengine's own `fakeReed` (`fakes_test.go`, which cannot import the kit without a cycle) gains `PasteSubmitCalls`, `PasteSubmitErr`, an `ActiveQueue []bool` consumed FIFO like `CaptureQueue` (last value sticks, empty reads not active) and `ActivityErr`.
`PasteSubmit` appends `paste:<guid>:<text>:<submit>` to `CallLog`; `OperatorActive` stays out of `CallLog`, like `SetWaitMark`, so the tests that pin today's exact call sequence keep passing while nothing calls it.

Tests: `TestReed_PaneTransportRecordsCalls` gains rows for both new ops' recording, and `TestReed_Overrides` rows for `PasteSubmitFn` and `OperatorActiveFn`.

**ImpactSummary:** Interface widened by two methods; its three implementers (reed's engine and the two fakes) all satisfy it in this card.
