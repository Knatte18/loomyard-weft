```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6, F7, F8]
disputed: []
deferred: [F8-lock-row, F7-episode-line]
```

## Changes

Both gates pass: tier 0 over the six touched packages, and tier 1 over `adoptrecord`, `fabricengine`, `beamcli` and `orchcli`.
The self-check changes were re-gated at tier 0 over `dobbyengine` and `adoptrecord`.

- **F1** (fixed as proposed, `3d9d25d0`): `ownerSubject` takes a `listed` func and returns the slug itself for a beam the record lists.
  `Covers` and the `adoptrecord` package doc state the rule.
  `TestWatchCovers` gains a row for a listed beam with no owner and one for a beam nested under a listed beam.
  The listed row fails with the defect in place.
- **F2** (fixed as proposed, `b1372568`): `TestArchiveOwnerTips` gains rows for origin ahead of the tip (no tag), a diverged origin (tag) and a branch absent on origin (tag).
- **F3** (fixed as proposed, `1689623e`): `internal/beamcli/ownerarchive_integration_test.go` pins the tags the archive makes for the prime (`root` and each run under it) and for a beam (only its own pair).
  It also pins that `ownerArchiveBeforeRefusal` archives nothing while a running adopt covers the worktree, and archives once the record is done.
- **F4** (fixed as proposed, `018e9ef5`, refined in the self-check commit): `TestRun_AnAdoptHoldSchedulesNothingUntilItLifts` spies on the pass spawn, task start, tidy, test-map and landing steps through two held ticks.
  The probe after the lift hands over: one archive, the run paused.
  A control test, `TestRun_ABeamTheHoldReleasedSchedulesAgain`, shows each spy fires once the owner leaves as the adopt lifts.
- **F5** (fixed as proposed, `018e9ef5`): `endArchiveEpisode` runs from `probeOwner` when no hand-over is due, and after a successful archive.
  `TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves` fails without it.
- **F6** (fixed as proposed, `3d9d25d0`, `6cfbed3d`): `TestWatchCovers` gains the nested-chain and `parentOf`-error rows.
  `TestWatcher_AnAdoptThatLiftsLetsTheNextTickStopTheStrand` covers the hold ending.
  `TestUnderAdoptWatch_CoversThePrimeWhileARootAdoptIsUnfinished` covers the accessor wiring over a hub.
  The once-per-episode failure log of `Watch` is not tested; see Deferred.
- **F7** (fixed as proposed, `018e9ef5`): `TestRun_ForeignOwner` gains rows for a done run, a run recorded owned elsewhere, and a failing run listing.
  `TestRun_AFailedBoltPullLeavesTheOwnerReadToTheLocalBolt` covers the failed pull.
- **F8** (fixed as proposed, `b1372568`): `TestArchiveOwnerTips_ATornPairDoesNotStopTheOthers` covers the joined failure over several pairs.
  `TestArchiveOwnerTips` asserts the origin branch tip is unchanged after the archive.

## Disputed

None.

## Deferred

- F8, held teardown lock row: `StatusPushLockWait` is a 30 s constant that no option or seam shortens, so a row would wait it out in real time, which the testing rules forbid.
  The fix is a production change to make the wait injectable, outside this finding.
- F7 and F6, once-per-episode log lines (`Watch`'s failure log, the Bolt pull episode): the lines go through the process-global logger, which the harnesses expose no seam for.
  The tests assert the behaviour around them, not the line.

## Rulings

None.

## Self-check

- fix-consistency, LOW: the `adoptrecord` package doc still described the old walk. Fixed in `doc.go`.
- fix-proposal, MEDIUM: the hold test had no positive control. Fixed with a control test over the same rig.
- fix-proposal, LOW: `endArchiveEpisode` logged a false message on the success path. Fixed: the message now differs per call site.
- fix-proposal, LOW: the Bolt pull test cannot observe its episode line. Deferred, same reason as above.
- fix-code-correctness, LOW: `endArchiveEpisode` message on the success path. Duplicate of the fix-proposal finding, fixed with it.
- fix-code-correctness, LOW: an unreadable owner ended the episode. Fixed: `probeOwner` keeps the episode while `ownerUnreadable` stands.
- fix-affected-set: no findings.

## Gate repair

The gate refused comment lines that wrapped at a fixed column or reached 120 characters, in `adoptrecord/doc.go`, `adoptrecord/watch.go` and `dobbyengine/handover_test.go`.
Each is now broken at sentence boundaries, one sentence per line, with the over-long lines split into shorter sentences.
No code changed.

The four lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.
