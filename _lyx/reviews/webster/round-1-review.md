---
verdict: APPROVED
---

Range: `git diff a35c8c2b1..HEAD` (`product.parent` is `main`; no rework round exists; no Publish failure on record).
`lyx gate test ./internal/shuttleengine/... ./internal/shedadapters` passes.

Verdicts on the round's focus questions, each grounded in the diff:

- **Card 4, backoff.**
  `awaitStartup` sleeps `base`, then doubles up to `ceiling = dismissalGap(cfg)`; `TestStartup_ReadyProbeTiming` pins 1, 2, 4, 8, 10, 10 s on the fake clock, so the ceiling is reached.
  Dismissals are throttled in `checkLivenessTick` by `dismissedAt` on the run's clock, and `TestStartup_GateNeverClearsUntilWindowExpires` pins key times at 0, 10, 20 and 30 s even when every probe sees a new capture (probes 1 s apart).
  A play whose first input fails does not start the gap; a later-key failure does.
  Every test injects the run clock; none waits out an interval.
- **Cards 3 and 4, `PATTERN-completion-signal`.**
  The diff adds no exit from `awaitStartup`; every negative answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or the direct `allOutputFilesExist`, and the held-dismissal branch falls through to `classifyStartupWindow`.
  `completionsignal_enforcement_test.go` is untouched.
- **Card 1, `PATTERN-shuttle-provider-seam`.**
  `shellEOLMark` and `holdsOnlyEOLMark` live in `claudeengine/session.go`; `shuttleengine` gains no Claude specifics.
  `%` beside other text and any other lone character stay drafts (rows in `TestIdleSession` and `TestInputBoxText`).
  `test-budget.yaml` is raised 103 to 104 for the one new top-level test, as the card names.
- **Card 2, `Result.StartError`.**
  `RunGated` is the only place that sets `NotStarted`, and it sets `StartError` beside it; the single-LLM producer appends the text after `ErrNotStarted` and keeps the existing text.
  The "died after a ready start" row pins that no other return carries it.
- **Card 3, pane tail.**
  `abandonStartup` appends `captureTail(run.lastStartupCapture)`, bounded by `paneTailLines`, only when `cause == nil`, `Started` is false and a capture exists.
  Rows pin the tail on the not-live path, its absence on mechanism failures, with no successful capture, after a ready start, and once-only when the cause already carries it.
- **Card 5.**
  `holdNoticeMaxDeferral` is 10 minutes; a search of code, docs, patterns and contracts finds no remaining 20-minute statement.
  `hold_test.go` now pins literal `10*time.Minute` and `15*time.Minute` bounds, which pins the value itself rather than following the constant.
- **Test economy.**
  The new rows extend existing tables (`TestIdleSession`, `TestInputBoxText`, `TestStartup_ReadyProbeTiming`, `TestStartup_RunGatedMappings`, `TestStartup_MechanismFailure`); the one new top-level test is the card's end-to-end check, which no other `claudeengine` test covers.
  Each added behavior has a covering row.

No finding.
