```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches behind F1, F5, F6 and F7 and checked that the fix covers every member.

Findings I left unchecked: F2, F3, F4, F8 and F9. None of their premises is a set of callers, sites or tests.

- **LOW**, card 9 Intent, the sentence "`cmd/lym`'s `testOnlyCodeAllowed` drops every entry cards 2, 3, 5 and 7 added".
  - Summary: card 8 already drops card 2's `FixHold`, `ClearHold` and `HoldRefusal.Error` entries, so card 9 can only drop what remains.
  - Evidence: card 8's Intent says it "drops the `internal/boardengine.FixHold`, `internal/boardengine.ClearHold` and `internal/boardengine.HoldRefusal.Error` entries card 2 added". F1's own fix says "removes every remaining entry". A reader of card 9 would look for three entries that are already gone; "every entry still standing from cards 2, 3, 5 and 7" fixes it.

Set coverage checked, no gaps:
- **F6:** every test file outside `internal/adoptrecord` that uses `adoptrecord` is covered by a card:
  - adoptcli: `TestAdoptIntegration_SecondCloneTakesOver` and `TestCommand_HereAdoptsTheSubjectOfItsWorktree` are edited in card 9. `TestAdopt`, `recordOf`, `world.workerDeps`, `TestRunAdopt`, `TestDiscard`, `TestWorkerArgs`, `stepsThrough` and `writeAdoptRecord` are edited or deleted in card 9.
  - orchcli: both `TestUnderAdoptWatch_*` tests are deleted in card 5.
  - beamcli: the owner-archive and dobby-deps tests are covered in card 5, the status and `TestAdoptSectionOf` tests in card 6.
- **F1:** card 6 makes nothing in `adoptrecord` unreachable. Every symbol beamcli reaches is also reached from adoptcli until card 9: `Read` and `WorkerAlive`, plus `Record.ReadState` (`caller.go:146`), `Record.ResumeStep` (`worker.go:143`) and `Request.Command` (`follow.go:45`).
- **F1:** `fixtries.Write` stays reached at card 8 through `deps.go:130` `ResetTries` until card 9 removes it, so card 8 needs no allowlist entry for it.
- **F5:** `hubGreenDeps` has one production caller, `check.go:497`, inside `hubCheckDeps`'s `Green`. Card 8 drops that seam and moves the call into `hubHoldCheckDeps`, which the fix changed to `hubGreenDeps(prime, subject, rerun)`.
- **F7:** the only call to `archiveOwnerPair` (through `ArchiveOwnerTips`) made under the moved slug's held lock is the one in `TakeOverRun`, and the fix reroutes it to `archiveOwnerSides`.

Tool calls:
1. Grepped the test files that still use `adoptrecord`, since my context held only the production importers.
2. Grepped the callers of `fixtries.Write`, `hubGreenDeps` and `greenEnd`, since my context lacked the test and `check.go` callers.
3. Listed the functions that use `adoptrecord` in each of those test files, to map them to the cards' Edit and Delete lists.
