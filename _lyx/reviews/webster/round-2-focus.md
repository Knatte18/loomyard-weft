---
round: 2
focus:
  - "In internal/shuttleengine/waitshadow.go, `logSessionState` passes `readSessionFacts` only the signals read since the shadow cursor. Settle whether the transcript interrupt marker (written after the `UserPromptSubmit` line) and the `ActivityReader` API-error marker are still read at a later liveness tick with no new bytes, so that the wait loop's fold agrees with the whole-file fold `lyx shuttle state` reports for the same session, and whether waitshadow_test.go pins a marker that appears after the turn start or turn end was folded."
  - "In internal/shuttleengine, check that comments written or changed by the diff name stable identifiers in backticks (for example `ParseSessionSignals`, `PATTERN-shuttle-provider-seam`) and never cite a plan card number or a plan decision, in claudeengine/signals.go and internal/shuttlecli/smoke_turns_test.go among others."
  - "In internal/shuttleengine/session_test.go, check that each test the diff adds calls `t.Parallel()` or carries a comment naming the global state that keeps it serial, per PATTERN-test-speed."
---

The focus lists where to look; it caps nothing.
