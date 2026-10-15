# Card 13 — model-switch-delete

**Delete:**
- `internal/shuttleengine#Engine.ModelSwitchSequence`
- `internal/shuttleengine#fakeEngine.ModelSwitchSequence`
- `internal/testkit/shuttlefake#Engine.ModelSwitchSequence`
- `internal/shuttleengine/claudeengine/modelswitch_test.go`

**Edit:**
- `internal/testkit/shuttlefake#Engine`
- `internal/testkit/shuttlefake#TestEngine_InertDefaults`
- `internal/testkit/shuttlefake#TestEngine_Overrides`
- `internal/shuttleengine#Config`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/claudeengine/startup.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Delete `ModelSwitchSequence` from the engine seam, every implementation and every fake (D7): no production code has called it since the builder module was deleted, and no board entry plans a live model switch.

Deleted: the `Engine` interface method, `Claude`'s implementation in `startup.go` with its doc comment, `fakeEngine`'s, `shuttlefake.Engine`'s with its `ModelSwitchSequenceFn` field, and `claudeengine/modelswitch_test.go`, whose one test pinned only that method.
`startup.go` is listed whole, since the card changes both its header and the method it deletes; its file header stops listing `ModelSwitchSequence` among its fixed key sequences.
`TestEngine_InertDefaults` and `TestEngine_Overrides` drop their `ModelSwitchSequence` assertions and the `ModelSwitchSequenceFn` override.
The `submit_settle_ms` template comment drops its `/model` clause ("and is the pause before the Enter of the /model command" and the matching "pauses not at all" wording), and `Config.SubmitSettleMS`'s line comment drops "and the pause before the /model command's Enter".
`claudeengine/doc.go`'s color paragraph ends at "with no leading Escape", dropping the comparison to `ModelSwitchSequence`.
`ColorSequence` stays on `Engine` (D7).
Lower `internal/shuttleengine/claudeengine` in `test-budget.yaml` from 113 to 112.

**ImpactSummary:** Removes one dead interface method with its three implementations and one test file; only comments and two fake tests change elsewhere.
