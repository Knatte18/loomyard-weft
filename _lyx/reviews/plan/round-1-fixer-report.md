# Plan-Review round 1 — fixer report

Every finding's premise held against the code, and every finding was fixed as proposed.
`lyx loom validate-plan` passes after the fixes, with only its two pre-existing informational `create-new-unit` notes on card 1.

## Changes

- **F1 (MEDIUM, fixed as proposed)** — `_lyx/plan/03-never-ready-pane-tail.md`.
  Verified: `skillReed.CapturePane` (`internal/shuttleengine/fakes_test.go:561`) returns only the sent texts, so the skill-load death row's startup probe records `""`.
  Verified: `TestStartup_MechanismFailure`'s rows fail or miss the strand before any capture (`flakyStatusReed` fails its first calls, `startup_test.go:49`).
  Card 3 now edits `fakes_test.go`.
  `skillReed` gains a `Screen` field that `CapturePane` serves first, and the death row sets it, so its no-tail assertion turns on `run.state.Started`.
  `flakyStatusReed` gains `passFirst`, and a new `TestStartup_MechanismFailure` row answers one live status with a non-empty pending capture before failing `maxStatusRetries` times.
  Every row of that table asserts no `The pane's last lines:`, so the cause condition is reached with the other two holding.
  No new top-level test.
- **F2 (LOW, fixed as proposed)** — `_lyx/plan/04-startup-probe-cadence.md`.
  The `TestStartup_GateDismissalGap` Create glyph is dropped.
  `TestStartup_UndismissableGateUntilWindowExpires` becomes the table-driven gate-never-clears test, renamed `TestStartup_GateNeverClearsUntilWindowExpires`.
  Its current case is the empty-sequence row, and card 4's three throttle rows join it.
  The package gains no top-level test.
  The old name survives only in `docs/benchmarks/test-redundancy.md`, a generated measurement report that no test reads.
- **F3 (LOW, fixed as proposed)** — `_lyx/plan/01-claude-lone-eol-mark.md`.
  The fake capture returns to `pane-idle-eol-mark.txt` once each send has landed, so the skill-load send's pre-write guard also reads the lone-`%` box.
- **F4 (NIT, fixed as proposed)** — `_lyx/plan/04-startup-probe-cadence.md`.
  `TestStartup_ReadyProbeTiming`'s `//testtiming:keep` reason is reworded to what the new rows pin.

## Disputed

None.

## Deferred

None.
