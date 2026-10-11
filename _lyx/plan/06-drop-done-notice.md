# Card 6 — drop-done-notice

**Edit:**
- `internal/battenshed#innerRunProducer.noticeCondition`
- `internal/battenshed#innerRunProducer.stateChangeReady`
- `internal/battenshed#innerRunProducer.noticeStep` -> `func (p *innerRunProducer) noticeStep(ctx context.Context, w *childWait)`
- `internal/battenshed#innerRunProducer.deliver` -> `func (p *innerRunProducer) deliver(ctx context.Context, key, condition, line string) bool`
- `internal/battenshed#innerRunProducer.wait`
- `internal/battenshed#innerRunProducer.check`
- `internal/battenshed#innerRunProducer.doneStep`
- `internal/battenshed#innerRunProducer.noticeDriverBusy`
- `internal/battenshed/doc.go`

**Delete:**
- `internal/battenshed#TestNotice_DoneChildNotifiesOnFirstSightAndTheDoneReturnAttemptsAPendingOne`

**Create:**
- `internal/battenshed#TestNotice_DoneChildSendsNoNotice`

**Intent:**
Run-Shed's `done` state-changed notice goes away (D4): a landing that succeeds needs nothing from the orch, and the Land and Worktree-Teardown rows (cards 7 to 9) notice the orch when a landing step fails or a landing readies a task.

`innerRunProducer.noticeCondition` returns no finding for a done child, and `innerRunProducer.stateChangeReady` loses its done branch.
The done arm's finish in `innerRunProducer.doneStep` no longer runs a last notice attempt, so the `final` parameter of `innerRunProducer.noticeStep` and `innerRunProducer.deliver` has no caller passing true and is removed, with its handling: the spacing bypass and the lost-notice Warn.
`innerRunProducer.wait`, `innerRunProducer.check`, `innerRunProducer.doneStep`, `innerRunProducer.noticeDriverBusy` and `innerRunProducer.noticeStep` are the callers updated to the new heads.
The done arm's wait for the driver's stop report is unchanged.
`internal/battenshed/doc.go` and `notice.go`'s header drop the done-notice sentences ("A done child's notice goes on the first check that sees it, and the Done return makes one last attempt of a notice still pending") and say a done child sends no notice.

Tests: `TestNotice_DoneChildNotifiesOnFirstSightAndTheDoneReturnAttemptsAPendingOne` asserted the removed behavior and is deleted;
the new `TestNotice_DoneChildSendsNoNotice`, built on `noticeHarness`, asserts a done child's checks and its Done return call Notify zero times and write no episode key, also when an earlier blocked notice of the same run was sent.
The test count of `internal/battenshed` is unchanged.

**ImpactSummary:** the done notice and the final-attempt flag leave battenshed's notice step; every caller of the two re-signed methods is in battenshed's own files.
