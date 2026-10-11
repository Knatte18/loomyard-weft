# Operator-quiet window reads the wall clock, not an injected clock

`operator-quiet-clock` · Note · enhancement, reed

Seen at pane-inject-safety PR-Gate (PR #548): `activeClientOnPane` in reedengine compares client activity against `time.Now()`, so tests of the `operator_quiet_s` window cannot use an injected clock.

Idea: take the clock from the caller, as the rest of the engine does, so the quiet-window edges are testable without sleeping.