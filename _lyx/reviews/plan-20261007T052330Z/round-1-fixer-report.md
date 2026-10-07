# Round 1 fixer report

Every finding in `round-1-review.md` is fixed in the plan files.
`lyx loom validate-plan` passes on the result, with one informational finding for Card 18's new unit.
`approved:` in `00-overview.md` is untouched, and the overview needed no change.

## Changed

- **F1** (`11-watcher-reloads-fresh-compaction-only.md`): a tick that reads a turn end sets `w.compactedAt` from its own evaluation alone and zeroes it under the first and third rules and when no boundary is found, so an earlier held boundary never reaches `tickIdle`; the held-boundary rule joins the `doc.go` rewrite, and the test list gains the held-then-stale row.
- **F2** (`13-orch-reload-sequence.md`): `tickResuming` handles the plugins step before its skills branch, and `typeReloadStep` gains a plugins case; the restart test now persists a `ReloadTypedAt` older than the handoff timeout with a turn end already read.
- **F3** (`14-loom-start-never-attaches.md`): `smoke_test.go` and `integration_driverbootstrap_test.go` join the Edit list, the two handover-evidence smoke tests are specified, the stale handshake comments in `bootstrap.go` are named, and the `ImpactSummary` is corrected.
- **F4** (`15-orch-start-never-attaches.md`): `pattern/PATTERN-cli-cobra.md` joins the Edit list with the sentence dropping `lyx orch start`.
  The same card's `doc.go` bullet also fixes the `DecideStart` outcomes sentence ("attach only, spawn a watcher, or relaunch") that the `StartAlreadyRunning` rename leaves stale; the review did not list that sentence.
- **F5** (`05-rerun-removes-superseded-strand.md`): the Intent states that probe-only callers inherit the teardown and why it is bounded, the test rows run through plain `Attach`, and the `ImpactSummary` names the callers.
  No new test row: `AttachGated` shares the body that plain `Attach` reaches, so the existing table already covers the probe-only entry.
- **F6** (`03-send-confirms-submission.md`, `02-claude-input-box-reader.md`): the running-turn row joins both tables, using `testdata/pane-turn-running.txt` for the reader.
- **F7**: `internal/shuttleengine/doc.go` joins Cards 3 and 5, `docs/overview.md` joins Card 4, and `internal/websterengine/doc.go` joins Cards 6 and 7, each with the sentence it must carry.
- **F8** (`08-step-error-keeps-handoff-voucher.md`): `internal/loomcli/arm.go` joins the Edit list, and `loomPostStep`'s comment joins the doc updates.
- **F9** (`16-orch-start-waits-for-stopping-watcher.md`): the `WaitWatcherGone` unit rows are cut, and `TestStart_LiveStrandAndWatcherDoesNothing` asserts the injected `sleep` is never called.
  This differs from the review's Fix, which kept a probe-once row in `watchloop_test.go` beside that assertion; keeping both would pin the no-wait case twice, so it is pinned once, at the verb.
- **F10** (`18-burler-validate-review-verb.md`): the separate verb test is folded into the parity table's rows, and the missing-file row is stated as `CheckReviewFile`'s wrapped read failure.
- **F11** (`02-claude-input-box-reader.md`): `submitRedrawSettle` is pinned at `300 * time.Millisecond`.

## Deferred

Nothing.

## Notes

Not a finding, kept for the next round: the review's claim that Card 8's cause is credible rests on reading `Shed.persistTransient` and `newCommitStatusSeam`, where a commit error is returned after the status file is written.
No reproduction was run, since job A writes no code.
