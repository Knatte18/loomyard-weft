# Lens review

All four forks returned a report.

- test-economy, MEDIUM, card 3's held-page row passes without card 3: fixed; the first page now uses 40 ms of a 50 ms `callTimeout` and page 2 answers after 30 ms, so a shared deadline fails the row. Merged with the matching row-reachability MEDIUM and criteria-coverage LOW.
- test-economy, LOW, card 2's `TestClassAndMark` deadline row repeats existing coverage: fixed; the row and the target are dropped, and the Intent cites `TestIsTransient`'s "wrapped url deadline" row, `TestClassAndMark`'s "github 503" row, `TestPRGate_ReturnedErrors` and `TestLoop_TransientRestepOnce`. Merged with the criteria-coverage LOW on the constructed error.
- test-economy, LOW, card 1's no-gate row duplicates an existing row: fixed; `held` zero is asserted inside the existing rows.
- test-economy, LOW, card 5's multi-line refusal row duplicates `TestQueueNotice_RefusesMultiLineAndSlashLeading`: fixed; the row is dropped.
- test-economy, LOW, card 10's lock check may not detect the record lock: rejected; `internal/state/state.go:137` takes `lock.AcquireWriteLock`, the same gofrs flock `lock.TryAcquireWriteLock` probes, which the criteria-coverage fork confirmed.
- forced-edits, LOW, the stale `Session.SessionState` comment: fixed; `internal/orchengine#Session` is on card 5 with the reworded comment.
- forced-edits, LOW, the `resume.go` and `resume_test.go` headers: fixed in card 10's Intent.
- forced-edits, LOW, card 7's platform files are not listed: fixed; `proc_linux.go` and `proc_windows.go` are targets.
- forced-edits, MEDIUM, card 4's unmatched-turn-start row cannot reach a `SessionBusyError`: fixed; a turn start after the API-error turn end folds the state to busy, so the state check is the turn-start refusal, and the row expects `ErrSendConditionUnmet`. Merged with the matching row-reachability MEDIUM.
- forced-edits, LOW, the `ActiveQueue` spelling is unchecked: rejected; `internal/shuttleengine/fakes_test.go:76` declares `ActiveQueue` on `fakeReed`, which `tuiReed` embeds.
- forced-edits, LOW, `cmd/lyx/pathrules_test.go:150` names "pause": no change needed, as the fork itself says.
- criteria-coverage, MEDIUM, no row for a cancellation after the deadline passed during the hold: fixed; card 2 gains that row.
- criteria-coverage, MEDIUM, the no-watcher revive report is not pinned: fixed; card 10's orch row runs with the watcher flag true and false.
- criteria-coverage, LOW, the error branches of status and revive have no row: fixed; card 9 gains a failed-readings row and card 10 a state-read failure row.
- criteria-coverage, LOW, shortcut forwarding is argued, not tested: rejected; forwarding is `internal/shortcutcli`'s generic routing, which its own tests pin, and card 8 pins the mapping through the index lines.
- criteria-coverage, LOW, the `cli_test.go` header still names `lyx pause --status`: fixed in card 8's Intent.
- criteria-coverage, LOW, the watcher check is tested with a stand-in, not `selfexec`'s seam: rejected; a test binary never execs, and `internal/selfexec#TestWatcher_Check` pins the exec seam itself.
- row-reachability, MEDIUM, a nil prime in `env.deps` would panic the existing rows: fixed; cards 9 and 10 set `Prime` only when `e.prime` is not nil.
- row-reachability, LOW, card 4's clean row needs a clock hook: fixed; the row uses `hookClock`, whose sleep hook appends the `START`.
- row-reachability, LOW, card 6's rows need a way to end `Run`: fixed; the `sleep` closure sets `fakeSession.alive` false after the third poll.
