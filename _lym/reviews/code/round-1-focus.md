---
round: 1
focus:
  - "internal/driftengine/capture.go `noFileReason` and `emptyUnderSkips`: does the reason name the scope, the count of skipped tests and that every test may have been skipped, as the Goal's Expected asks, and is `req.Skip` the same list the scope record and `capture.Skip` hold when the capture ran under a non-partial scope?"
  - "internal/driftengine/await.go and baseline.go: does every capture Stuck that follows a wrote-nothing capture under skips name `lym drift scope --scope full|off` (or a shorter `--skip` list) before `lym drift capture` and `lym loom resume`, besides `lym drift defer`? Settle whether `rescopeWaysForward(true)` in `captureAgain` is sound for every path that reaches it, and whether the `onBranchPoint` gating in Drift-Await matches the Goal."
  - "internal/driftengine/records.go `WroteNoFile`: is a capture record written before this change, or one read back through `internal/state`, handled without a wrong rescope Stuck or a lost field?"
  - "Scope against the Goal: does the diff change anything the entry did not ask for, such as the plain-capture way forward for a capture that failed for another reason?"
  - "Test economy in await_test.go, baseline_test.go and capture_test.go: is each new test a distinct behavior, and is any behavior the diff adds (the single-skip wording, the not-on-branch-point branch, the Drift-Baseline Stuck) left uncovered?"
  - "doc.go and the commit message: does each claim bear out against the diff, and do the doc.go lines follow the repository's own comment conventions?"
---

The decision record holds only the board entry under `## Goal`, so the Goal's `Expected` list is the measuring stick.
