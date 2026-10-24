---
round: 1
focus:
  - "internal/gatecli/run.go `report` and `runEnd.halt`: settle whether `halt` is set exactly for a refusal, an acquire failure and a signal-cancelled context, and never for a timed-out tier or an empty-tier failure, which the decision record says count as failed runs; check what the per-tier timeout context is, against `e.ctx`."
  - "internal/gatecli/run.go `runTest` loop: settle whether the exit code on a halt (`exitAfterSignal(caught, end.code)`) matches the old behavior for a refusal (1), a busy hub (SlotBusyExit) and a signal (128 plus the number), including when `end.code` is 0 after a cancelled context."
  - "internal/gatecli/run.go `runGreenAware`: settle whether `end == (runEnd{})` still records a pass only for a clean run, so a failed tier never carries (`PATTERN-gate-carry`), and whether a carried or empty tier still reports exit 0 without halting."
  - "internal/gatecli/run.go closing line: settle whether it prints only when a run failed, names every failed listed and guard run with its exit code, goes to stderr, and leaves a refusal's stdout a single JSON envelope."
  - "The guard step now runs through the same loop as the listed tiers: settle whether plan order really puts guard runs last, whether `runGuard`'s error path (`guardStepRun` failing) still ends the call as before, and whether removing `guardRunOf` leaves no caller or stale comment."
  - "internal/gatecli/cli.go, doc.go and docs/overview.md: settle whether every sentence still saying the call stops at the first failing tier or skips the guard step after a failure is updated, whether the `test` Long text stays within the CLI-form test's byte cap, and whether the exit-code contract text matches the first-failing-run rule."
  - "internal/gatecli/test_integration_test.go and scopes_integration_test.go: settle whether the changed tests cover the acceptance criteria (first tier fails and a later tier runs, a different later failure keeps the first code, guard step runs after a listed failure, the closing line, and none when all pass) and whether the refusal, busy-hub and signal stops are still pinned; flag duplicated coverage by naming the covering test."
---

Seed focus from the board entry, the decision record and the diff of `run.go`.
