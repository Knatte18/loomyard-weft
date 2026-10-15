---
round: 1
focus:
  - "internal/shuttleengine/run.go, send.go, readiness.go: the task's recordless paths (`Interrupt`, `Send`, `recordlessSendContext`, `sessionReadiness` with `recordless`) merged with the parent's per-run engine resolution (`engineFor`, `EngineSet`). Settle whether every recordless path picks the default engine the same way and still skips run-record-only steps (pending-send record, run-state engine lookup), and whether `InterruptSettle`, `PaneRunningTurn` and any other `FindRun` caller still refuse or handle a strand with no run record consistently with `Send` and `Interrupt`."
  - "internal/creelcli/delivery.go and its tests: creel's `Runner` construction and idle, send and interrupt calls against the parent's new `NewRunner(reed, engines, ...)` signature and `enginesFor` seam. Settle whether the adapter follows `PATTERN-shuttle-provider-seam` (only `internal/shuttleengines` constructs or imports a concrete engine) and whether the hub-wide set is built once per worktree runner as the registry expects."
  - "internal/testkit/shuttlefake and internal/testkit/creelfake: the parent changed `shuttlefake`; creel's tests and `creelfake` rely on it. Settle whether the fake's changed behavior (engine recording, signatures) still serves creel's recordless idle, send and interrupt tests, and whether any creel test is now redundant with a parent test or leaves the recordless path uncovered (`PATTERN-test-economy`)."
  - "PATTERN.md, pattern/PATTERN-told-geometry.md, pattern/PATTERN-pane-text-injection.md, pattern/PATTERN-shuttle-provider-seam.md: both sides edited them. Settle whether creel's new code (`creelengine`, `creelcli`, `hubgeom/creelgeom.go`, `selfexec`, `proc`) follows each changed invariant, and whether the merged text of the shared pattern files still states both sides' rules without one overwriting the other."
  - "cmd/lyx/testdata/test-budget.yaml and docs/overview.md: both sides changed rows. Settle whether the merged budget rows and module table still match the merged tree (creel packages and the parent's new `shuttleengines` package both present)."
  - "internal/reedengine/lifecycle.go and internal/shuttlecli/send.go, interrupt.go: both sides touched them. Settle whether the task's edits and the parent's registry changes still agree on the `Runner` and reed calls they make."
---

Seeded from the overlap list and a read of the shuttle diffs.
The merged `run.go` and `send.go` already resolve the engine through `engineFor` on the recordless paths, so the first entry asks for the remaining sites rather than a known defect.
