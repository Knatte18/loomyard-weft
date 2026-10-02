# Webster round 1 — fixer report

All four findings are fixed, one commit each.

## Fixed

- **F1 (LOW)**, `2f0e417c0` `battenshed: fix F1 — a child seen out of a halted state ends the halt episode`.
  `innerRunProducer.Call` now removes the halt-warned marker whenever the child is not blocked, paused or failed, beside the existing done-seen clear; a remove failure other than not-exist is a hard error.
  `Call`'s and `callHalted`'s doc comments say an episode ends on an observed resume or a changed history length.
  New test `TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength` drives failed (length 4), running, failed (length 4) and asserts two Warns.
- **F2 (LOW)**, `962eb6462` `shuttleengine: fix F2 — refuse a terminal result from a pass_on_cap entry`.
  `evaluateGate` returns a gate error for a `Terminal` failing result from a `PassOnCap` entry, before writing findings, so it stores no memo.
  `GateResult.Terminal`'s and `evaluateGate`'s doc comments state that only a must-pass entry may return it.
  `TestGateTerminal_TerminalOnPassedOrPendingOrPassOnCapIsGateError` gains the pass_on_cap case.
- **F3 (LOW)**, `539aff107` `parentreview: fix F3 — GateConfig.Reviewer names the resolved parent, not the seed`.
- **F4 (LOW)**, `f92381c78` `fabricengine, battencli: fix F4 — semantic line breaks in the new comments`.
  Rejoined the fixed-column-wrapped new comments in `origin.go`, `origin_test.go`, `origin_integration_test.go` and `lifecycle_integration_test.go`; legacy lines around them are untouched.

## Verification

`go build ./...` and `go vet -tags smoke ./...` are clean, and `go test -tags integration ./...` passes over the fixed tree.

## Deferred

None.
