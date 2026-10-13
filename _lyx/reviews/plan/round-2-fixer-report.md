# Plan round 2 — fixer report

## Changes

- **F1 (BLOCKING, design)** — fixed, one line fixed differently.
  The D11 table's lead paragraph now says the sweep follows each loop's per-iteration call graph, including hooks and callbacks handed to another package's polling loop (`Spec.Loop.OnWake`, `Waiter.Paused`), and a site reached that way names the loop that reaches it.
  Re-running the sweep that way added rows for every line the review named, plus one it did not: `shuttleengine` `Run.watchDialog`'s `capture pane for the dialog watch failed`, which repeats on every wait tick while a dialog latch stands and the capture fails (guard, in card 11).
  It also added keep rows for the callees and callbacks it checked and found already once per change, event or attempt: `runResizePinsLocked`, `repairNames`' `repaired pane title`, `logSessionState`/`showWait`, the `shedverbs` loop step's Warns, `loopLockExit`, `parentreview`'s `pollAlignment` and `parentGuard.ensure`'s `OnWait` lines.
  New cards, inserted with `lyx loom insert-card`:
  card 4 `reed-callee-episodes` (`liveBoxLocked`, `readBorderTitleRowLocked`, `keepZoomLocked`, `watchNames`, `repairNames`, with `TestCalleeFailuresLogOncePerEpisode` reaching each site's Warn, Debug and Info-recovery lines);
  card 14 `loomcli-idle-release-guards` (`idleReleaseOn`, `ownedLockBranches` re-signed to return its error, `loopLockExit`, with `TestIdleReleaseLogLevels`);
  card 15 `landingshed-pause-read-guard` (`parentGuard.pauseRequested` taking an episode that `ensure` holds as a local, with `TestParentGuard_PauseReadFailureLogsOncePerEpisode`).
  The renumber moved the old cards 4–16 up; the table's Card column and the three stale prose mentions it reported (`00-overview.md` lines naming cards 16 and 4, and `18-land-notice.md`'s card 16) now name the new numbers.
  **Fixed differently:** the review asked to guard `loomcli: could not release the idle parent lock`.
  It does not repeat per wake: `idleReleaseOn` resets `watch` to a fresh `parentlock.IdleWatch` after every release pass, failed or not (`internal/loomcli/looplock.go`, the `watch = &parentlock.IdleWatch{Bound: bound}` after the release loop), so the next attempt comes only after another full idle bound.
  It is listed as keep with that reason, and card 14 says so.
  The other three loomcli lines are guarded or demoted as proposed.
  `parentGuard` has value receivers, so card 15 holds the episode as a local in `ensure` behind a closure, which the fix allowed.

- **F2 (MEDIUM, scope)** — fixed as proposed, in card 11 (formerly card 10).
  (a) `TestGatePending_FailedSendWarnsAndStaysPending` scripts four pending evaluations and a per-tick step that clears `f.reed.SendTextErr` once two pending sends failed, and asserts one WARN, at least one DEBUG and one INFO `gate: pending send delivered again` line.
  (b) `fakeReed` gains `SendKeyErrQueue`, and `TestRun_Wait_StartupWindow` gains the row "trust dismissal fails on the first probes then sends", asserting one WARN `dismiss trust prompt` and one INFO `trust prompt dismissal sends again` line.
  The Intent now defines a dismissal that sends as `playInputsSent` returning no error for a non-empty sequence; an error with `sent > 0` is a failure.
  Card 11 also carries the new `watchDialog` guard, reached by `TestWait_DialogNotice`'s existing failed-capture row, which now asserts one WARN and one INFO recovery line.

`lyx loom validate-plan` reports `ok: true`, with only informational new-unit findings.

## Disputed

None.

## Deferred

None.
