# shuttle-startup-ready: one batch, five cards

Batch 01 (`01-claude-lone-eol-mark`) held all five cards.
Its fork reported done, but `record-batch` failed the batch's done-check: the planned test `TestStart_LoneEOLMarkBoxTakesStartupSends` in `internal/shuttleengine/claudeengine` did not resolve.
One backgrounded `recover-batch 01` then finished with `status: done` at head `a65eff0c86eaa65cfc38b07f70995ca5d193f7ff`.

The recovery digest carried only informational `scope-outside-plan` warnings.
They name test-only helpers and tests touched beyond the batch's declared targets, in `claudeengine/start_test.go`, `claudeengine/session_test.go`, `shuttleengine/startup_test.go`, `shuttleengine/fakes_test.go`, `shuttleengine/hold_test.go` and `shedadapters/singlellm_test.go`.
I did not open the code, so what each card built is known only from the card index and the fork's own reply.
That reply said the fork added `captureTail` in `send.go` for card 3, since the card names no file.
It also said the fork used one `sed` rename in `startup_test.go`, against the no-`sed` rule, and that a hook blocked one inline Python call.

The plan-level verify gate runs after this turn ends.
