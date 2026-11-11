```yaml
lens: fix-proposal
outcome: reported
```

Findings for the fix-proposal lens, in order of severity.

**1. MEDIUM — `TestRun_AnAdoptHoldSchedulesNothingUntilItLifts` has no positive control.**
- The test asserts every spy stays at zero while the hold stands, but it never runs with the hold absent. Removing the `!d.adoptHolds` gate from `landBeam` (`internal/dobbyengine/loop.go:673`) could therefore leave it passing.
- `landBeam` returns early unless `CanLand` is set and `geom.Slug` is non-empty (`land.go:32`), and the test sets both. The test then returns `LandReading{}` from `CanLand`.
  - If a zero `LandReading` leads to an early return after the read, a missing gate still shows up as `landReads > 0`, so the landing gate is covered.
  - Whether a zero reading and a beam dobby actually reach the tidy and test-map steps is unproven.
- The tidy and test-map steps only fire when their conditions are met. `testMapStep` needs `LandingsSinceTestMap >= TestMapEvery`, which comes from counting the fake `Landings` reads. The tidy step has its own `BoardTidyEvery` count, and I did not read its conditions.
- The only control for these conditions is `TestRun_NoPassWhileAMoveStands`, which uses a prime dobby with no `Slug`. This test has `Slug` set, so a different early return could keep the tidy or test-map spy at zero for a reason other than the hold.
- The reviewer's proposal said to add spies and expect none to run. Carried out literally, that proposal can pass vacuously.
- Evidence: the test lifts the hold and hands over, so no tick ever runs unheld. A second tick with the owner cleared would show each spy firing.

**2. LOW — `endArchiveEpisode` is called on the successful-archive path, and its message misdescribes that case.**
- At `handover.go`, `handOver` calls `endArchiveEpisode` after a successful `ArchiveOwner`. The log message reads "the beam needs no hand-over any more", which is false there, because the archive just succeeded and the dobby exits for the hand-over.
- This follows the reviewer's "also reset after a successful archive" proposal. The result is a misleading Info line, and only when an earlier failure episode was open.
- It has no behavioural effect, since the dobby exits right after.

**3. LOW — `TestRun_AFailedBoltPullLeavesTheOwnerReadToTheLocalBolt` does not check the proposal's "one episode line".**
- The test asserts `pulls == 1` and the hand-over. The episode line goes through `logger.Episode`, which the test cannot observe.
- A broken `boltPull.Fail` call would still pass.

I found nothing wrong in the F1 proposal itself. Checking the listed slug before the owner test returns the same subject as before for any beam that has an owner. It only changes the case of a beam with no owner that the record lists.

Further tool calls (2):
- `grep` on `landBeam`, `tidyStep` and `testMapStep` in `internal/dobbyengine`, to see which early returns the held-tick spies depend on. My context did not show those functions.
- `sed -n 10,45p testmap.go`, to read `testMapStep`'s firing conditions.
