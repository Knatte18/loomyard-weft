# Fixer-report, code review round 1

F2 is a `decision`-class finding and went to the parent, who recorded that the verb stays detached; it is left out below.

## Changes

- **F1** (`internal/driftcli/capture_integration_test.go`), fixed as proposed.
  The capture-failure subtest no longer requires `env.OK`.
  It accepts either an OK envelope or a not-OK one that names the capture's failure (`drift: the baseline capture ended failed (exit 1; it ran in `, `printed nothing`, a log, the `lym drift capture followed by lym loom resume` way forward).
  It keeps the `waitFor` on the record reading failed.
  A deterministic row reads the one `capture-child-*.log` and asserts its last line is an envelope with `ok` false and an error opening with the record's reason.
- **F3** (`internal/driftengine/steprun.go`, `capture_test.go`), fixed as proposed, with one addition.
  `commandReport` reads only the last 64 KiB of the log through `readLogTail`, and quotes an over-long tail from its start behind a leading ellipsis through `boundOneLineKeepingEnd`; `readChildReport` still uses `boundOneLine`.
  The over-long row now expects the ellipsis first, and two rows are new: a short last line after two long lines, and a log past the read window.
  The self-check added reading from one byte before the window, so a window that begins on a line start keeps its first line.
- **F4** (`internal/orchcli`), fixed with a small production extraction.
  `InstallReconcileNotifier` now installs `reconcileNotifierFor(location)`.
  The proposed rows called the notifier `hubreconcile` holds, which the package keeps private, so the test calls the extracted builder.
  `TestReconcileNotifierFor` covers the hub's own notice mailing the prime orch from `RECONCILE` and another hub's notice returning false with nothing posted.
  `TestRootPreRun_InstallsTheHubNotifiers` reads the root pre-run source for both install calls; no earlier wiring test covered the push notifier there either, so it is new.
  The creel setup and the posted-mail read of the push notifier test became `startHubOrchDaemon` and `postedMails`, which both tests use.
  The proposed "install from outside a hub, assert no notifier changes" row is not written: the installed notifier cannot be read back, and the hub gate is the `hubLocation` call the push notifier's outside-a-hub test already pins.
- **F5** (`internal/driftengine/dirbase.go`, `dirbase_test.go`), fixed as proposed.
  A dir node tagged `!!null` is skipped, and `NullDirsAreKept` covers `dir: null` and `dir: ~`.
- **F6** (`internal/driftengine/capture_test.go`, `compare_test.go`), fixed as proposed.
  A main-capture timeout row, the accepted-capture timeout row and the compare timeout assertion name the directory and the output.
  The missing-dir rows of "the step's dir" fail the test if `OnEnd` is called.
  The unreadable-log clause has a row: the command removes its own log, and the reason holds `its log could not be read: `.
- **F7** (`internal/hubreconcile/reinterpret_integration_test.go`), fixed in part.
  Rows for a notifier reporting not posted, a notifier error and no installed notifier assert `Ensure` returns nil, the marker is committed and the file keeps its flagged dir.
  The no-notifier row relies on every earlier test having restored a nil notifier; it cannot detect one that leaked.
  The `Rewrite`-error row is deferred below.
- **F8** (`internal/configengine/config.go`), fixed as proposed.
  The `ReinterpretedMarkerRel` comment stands alone.

Each fix is its own commit, except F3 and F6, which share `capture_test.go` and went in one commit.

## Disputed

Nothing.

## Deferred

- **F7, the `Rewrite` error that fails the walk.**
  The row the review proposed (a board `drift.yaml` of `capture: {dir: [`) does not reach the `fileError` branch of `applyReinterpretations`.
  I wrote it, and the gate failed on my assertion for the reinterpretation's wording: the error began `hubreconcile: reconcile config in …/_bolt: …`, which is the reconcile's own parse failing before `applyReinterpretations` runs.
  The same file cannot parse for the reconcile and fail in `Rewrite`, and `configreg.Modules()` is a fixed registry, so reaching the branch needs a production seam to register a failing reinterpretation.
  A seam is a change outside the finding's files, so I removed the redundant test instead.

## Self-check

- fix-consistency, LOW, `outputTailMaxRunes` comment did not say the tail keeps its end: fixed as proposed.
- fix-consistency, LOW, `readChildReport` cuts from the end: no change, a deliberate asymmetry the review sets.
- fix-proposal, MEDIUM, the F7 row does not reach the `Rewrite` branch: fixed with my own change; the test is removed and the branch is deferred above.
- fix-proposal, LOW, the no-notifier row cannot detect a leaked notifier: noted in the F7 entry; no seam is added.
- fix-proposal, LOW, `readLogTail` dropped the first line when the window began on a line start, and no test passed a log over the window: fixed as proposed; it reads one byte before the window, and a row covers a log past it.
- fix-code-correctness, LOW, the F7 row does not pin the `Rewrite` branch: the same as the fix-proposal MEDIUM above.
- fix-code-correctness, LOW, `readLogTail` lacked a test over the window and had two edge behaviours: the window test is added, the line-start case is fixed, and a window with no newline keeps its partial line, which only affects the quoted tail.
- fix-affected-set: returned no findings.

The lens reports are beside this file as `round-1-fixer-report.lens-<lens>.md`.
