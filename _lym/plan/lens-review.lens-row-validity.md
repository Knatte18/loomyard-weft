```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked each planned test row and each new test against the existing tests and fakes the cards name.

**Findings**

1. **BLOCKING, card 10, `TestBattenRun_RefusesWhileAPauseCoversTheSlug`.** After card 10, the test's covering rows no longer refuse.
   - Card 10 moves the pause check from `refuseCovered` into `refuseByAct` and says a nil `runVerdict` checks nothing.
   - The test drives `newFakeReceiver` through `runCommand(c)`, which calls `c.driveRun` directly and never reaches `armAt` (`internal/battencli/run_test.go:203-205`). Its `runVerdict` is therefore nil.
   - The "an entry for the slug refuses" and "a hub-wide entry refuses" rows would seed and run (`internal/battencli/pausehold_test.go:28-70`).
   - Card 10 asserts the act rule's text there but never says the test sets a fake `runVerdict`, or which one.

2. **BLOCKING, cards 8 and 11, `internal/battencli#TestDobbyReading`.** The rows these cards add to this test cannot reach `DobbyRuns`.
   - `TestDobbyReading` only exercises `dobbyStateReading` and `awakeReading` (`internal/battencli/dobbyreading_test.go:23-72`).
   - Card 8's row ("an unfinished intent setting `CancelMarker`") and card 11's rows ("`Run` and `Verdict`") need `DobbyRuns`.
   - The only `DobbyRuns` callers in tests are tier1 tests in `internal/beamcli/dobbypairanchor_test.go:96,127,157,227`. Neither card lists them.
   - Card 11 makes `DobbyRuns` call `pairteardown.ReadRun` and drop a run whose read fails. Those beamcli tests therefore become forced edits, and could break if `ReadRun` errors over their hub, for example on owners or fabric config.

3. **BLOCKING, card 9, `internal/pairteardown#Teardown.Run` and `TestTeardown_CallSequence`.** Existing `Run` rows would panic.
   - The fake builds `Teardown` as a literal in `fakeSubstrates.teardown` (`internal/pairteardown/teardown_test.go:111-150`).
   - The new field `removeCancelledRecord` would be nil there, and card 9 has `Run` call it after every successful removal.
   - Card 9 states no meaning for a nil field and does not list the helper `fakeSubstrates.teardown` that must fill it.

4. **MEDIUM, card 8, `internal/pairteardown#New`.** The same fake sets `cancelMarked` (`teardown_test.go:116`).
   - Card 8 renames the field to `cancelling`, so `fakeSubstrates.teardown` must change.
   - Card 8 lists `TestRegisteredBattenAlive_ReadsADobbyDrivenRun` but not the helper. The build breaks until the helper is edited.

5. **MEDIUM, card 5, `TestReadRun`, D3 criterion "a run whose `.lym` is removed reads the same FSM state".** The planned row cannot fail if the code is wrong.
   - The row runs over fakes and "deletes every `.lym` file the fakes stand for". A fake reads no file, so deleting files changes nothing it returns.
   - Card 3's `TestDerive` "with no cache" is vacuous for the same reason: `Derive` takes `Records` only.
   - So no planned row would catch a real reader that consults `.lym`. The criterion needs a row over real files, for example `ReadRun` over a temp hub with its scratch removed, or a tier1 row.

6. **MEDIUM, card 16, `TestMoveScratchAttempts`.** This tests an unexported helper rather than its surface.
   - The existing parent-align fixer test already covers a one-time migration of a retired attempts file (`seedRetiredAttempts` at `internal/landingshed/parentalign_fixer_test.go:572`, and the migration at `parentalign.go:324-339`).
   - The scratch-to-records move belongs as rows there and in the Finalize attempts tests, reached through `ParentAlign.readAttempts` and `Finalize.readAttempts`. A new top-level test of the helper breaks the test-economy rule.

7. **LOW, card 10, `TestBattenPreRun_AsksTheActRule`.** This new test covers `battenPreRun`, which already has `TestBattenPreRun_ContinuePass` (`continue_pass_test.go:32`) over the same hand-populated receiver. The act-rule rows fit as rows of that test or of the pause-hold test, which card 10 rewrites anyway (see finding 1).

8. **LOW, card 7, `TestCancelRun`.** No default is named for the new seams in existing rows.
   - `cancelDeps` is documented as "None is nil except BeamProject and LandingHeld" (`cancel.go:31`).
   - Card 7 adds four seams: `Verdict`, `WriteIntent`, `CommitRecord` and `FinishCancel`. Every existing `TestCancelRun` row would call them as nil unless the test's deps builder defaults them, and the card names neither a default nor the builder.
   - Its "intent written and committed before `EndForCancel`" row also needs a call-order recorder across `WriteIntent`, `CommitRecord` and `EndForCancel`. The card does not say the fakes record order.

9. **LOW, card 9, `cleanupDeps.PairPresent`.** No nil meaning is stated. `TestCleanup` and `TestClearDoneRun_*` build `cleanupDeps` by hand (`cleanup_test.go:161`, `cleanup_test.go:557`), and a cancelled row reached with a nil probe would panic.

**Tool calls made beyond my context**
- One grep for the named tests' locations and tiers: my context did not have the test files.
- `run_persist_test.go` and the persist code in `run.go`, to check whether Shed's persist carries fields an outside writer added (it does, through a mutate on the current status).
- `TestDobbyReading`, the pause-hold test and the setup of `TestRunCmd_ResumeDispositions`, to see what each actually drives.
- `newFakeReceiver` and `runCommand`, which `DobbyRuns` test callers exist, and how the teardown fake is built.
- The teardown fake literal, the `DobbyRuns` tests in beamcli, the shape of `TestPause`, and the existing attempts-migration test.
