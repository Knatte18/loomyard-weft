---
verdict: APPROVED
rationale: "Review range main merge-base 812600b3a..HEAD is correct (product.parent main, no rework round); the round-1 review's four LOW findings are real and each fix landed (2f0e417c0, 962eb6462, 539aff107, f92381c78) with tests, and build plus battenshed, shuttleengine, parentreview, fabricengine, loomcli and hubgeom tests pass."
---

The review range is right: `_lyx/shed/run-parentage/status.json` carries `product.parent: main`, `_lyx/loom/rework/` does not exist, so the subject is `git diff $(git merge-base main HEAD)..HEAD` from `812600b3a`.

The report carries no `## Focus departures` section; its seven focus verdicts answer the seven round-1 focus entries one for one, and their not-a-defect reasoning holds against the code.

Each finding is real and fixed:

- **F1** (`internal/battenshed/innerrun.go`): `Call` now removes the halt-warned marker whenever the child is not blocked, paused or failed, with a non-not-exist remove failure as a hard error, and `callHalted`'s doc no longer claims every re-halt grows the history.
  `TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength` drives failed, running, failed at length 4 and asserts two Warns.
- **F2** (`internal/shuttleengine/wait.go`): the pass_on_cap guard sits after the pending and passed arms, so it fires only on a failing result and before the findings file is written.
  `GateResult.Terminal`'s and `evaluateGate`'s docs state the rule, and `gate_terminal_test.go` gains the case.
- **F3** (`internal/parentreview/gate.go`): `GateConfig.Reviewer` names the parent the wiring resolves.
- **F4**: the listed comments are rejoined to one sentence per line.
  A scan of every added Go comment line in the range finds the remaining multi-line comments break only at clause boundaries, as CONSTRAINTS.md permits.

`go build ./...` is clean, and `go test` passes for every package the fixes touch plus `loomcli` and `hubgeom`.
