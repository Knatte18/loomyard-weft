# Plan-Review round 2 — fixer report

Every finding was checked against the code and the decision record, and every one held.
All four are fixed as proposed.
After the edits, `lyx loom validate-plan` reports only the informational `create-new-unit` findings.

## Fixed

- **F1** (the lease test cannot see a release and does not pin the lease's start): the premise holds.
  `AcquireStateMutation` returns a `*lock.FileLock` (`internal/websterengine/state.go:103`), and `Release` is a method on that concrete struct (`internal/lock/lock.go:76`), so a wrapper cannot intercept it.
  In card 4, `renumberDeps.acquireLease` is now typed `func() (release func() error, err error)`, and `newRenumberDeps` fills it from `AcquireStateMutation` and the acquired lock's `Release`.
  The Intent states that the lease is taken before the run record or the plan directory is read.
  In `TestRenumberVerbs`, the test wraps the returned release, and the wrapped `loadState` fails the test when it is called without the lease, the same as an `ops` call.
  The checks run on the verb's own call path, with no goroutine.
- **F2** (the stated bounds omit writes outside the plan directory): card 4's insert/remove bound now carries D2's "in the plan directory" qualifier.
  It also names the writes outside that directory: the journal's parent directory, the temporary journal and its rename to `planparser.RenumberJournalPath`, the journal's removal, and webster's scratch directory and lock file, which the lease creates.
  The restore bound now says that outside the plan directory it removes only the journal, and that it removes the journal last.
  The restore cost statement now says it removes every journaled created name, whatever that file holds by then.
- **F3** (the reversal does not say how it handles names not yet created): the premise holds, because `os.Remove` fails on an absent name.
  Card 2's `reverseRenumber` sentence now says it skips a `Created` name that does not exist (`fs.ErrNotExist`) instead of failing.
  Card 3's idempotency sentence now cites that rule.
- **F4** (an unreadable journal has no outcome in restore): card 3's `readRenumberJournal` now returns any read error other than `fs.ErrNotExist` as `ErrRenumberIOFailed` wrapping the cause, with nothing written, so it joins the loom raw I/O row that card 4 widens.
  `TestRestoreRenumber` gains that row, which is a row in an existing table and needs no budget change.
  Card 3 lists `plan:internal/planparser#ErrRenumberIOFailed` under `**Uses:**`.

## Disputed

None.

## Deferred

None.
