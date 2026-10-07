# Card 1 — paced-text-submit

**Edit:**
- `internal/shuttleengine/claudeengine/startup.go`
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/startup_test.go`
- `internal/shuttleengine/claudeengine/modelswitch_test.go`
- `internal/shuttleengine/claudeengine/session_test.go`

**Intent:** D2: every Claude key sequence that types text and submits it types the text unsubmitted, settles, then sends Enter as its own key step.
Claude Code reads a fast character burst as a paste and swallows an Enter that arrives inside it, so the settle moves the Enter out of the burst.

Add one provider-owned constant in `startup.go`, `submitSettleMS = 300`, with a comment naming the paste window it clears (A5).
Rewrite the four text-plus-submit sequences:

- `Claude.ComposeSend`: `{Key: "Escape", SettleMS: composeSendSettleMS}`, `{Text: text, SettleMS: submitSettleMS}`, `{Key: "Enter"}`.
- `Claude.ModelSwitchSequence`: `{Text: "/model " + model, SettleMS: submitSettleMS}`, `{Key: "Enter"}`, still with no leading Escape.
- `Claude.ClearSessionSequence` and `Claude.CompactSessionSequence` in `session.go`: the same text-then-Enter pair, still with no leading Escape.

`reedengine.SendText`'s transport contract stays unchanged: the text step passes `Submit: false`, and `playInputs` already honours `SettleMS` after each step through `inputSleep`.
Update each sequence's doc comment to say the text and the Enter are two paced steps and why.

Tests: `TestComposeSend` in `startup_test.go`, the model-switch test in `modelswitch_test.go` and `TestCompactSessionSequence` plus the clear-sequence assertion in `session_test.go` pin the exact three- or two-step slices, with `Submit` false on every text step, the settle on the text step and a bare `Enter` key step last.

**ImpactSummary:** Four claudeengine key sequences change shape; every caller plays them through playInputs unchanged, and no other engine is touched.
