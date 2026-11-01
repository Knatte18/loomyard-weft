```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it.

**Per behaviour: pinning test**

1. **A dead driver inside the grace sends no notice** (`runnotice.go:251`).
   - Pinned by `TestRunNotice_Conditions` / `AJustStartedChildWithNoDriverLockYetSendsNothing`.
   - It advances `driverStartGrace/2` with `alive=false` and expects no notice.
   - Removing the grace branch makes it fail.
   - It is not a boundary test: nothing runs at exactly `driverStartGrace`.
   - The moved-past-grace rows do pin the `&gt;=` side, since they advance exactly `driverStartGrace`.
   - The `h.advance` call and the harness `tick` both move the clock, so the awake time is a little over the grace. A flip from `&lt;` to `&lt;=` would therefore pass undetected, and I don't count that as a finding.

2. **A dead driver past the grace still sends the notice.**
   - Pinned by the two existing dead-driver rows (`ADeadDriverReadsTheChildsStateAndReasonRelativeToThePair`, `ADeadDriverAtTheRespawnCapNamesTheCapAndLoomStart`).
   - It is also pinned by `TestRunNotice_OncePerEpisode` and `TestRunNotice_DeliveryIsRetriedOnProbeTicksThenDropped`, which now advance the grace first.

3. **Machine sleep does not count toward the grace** (`AwakeSince(..., sleeps, ...)`).
   - Pinned by `AMachineSleepDoesNotCountTowardTheStartGrace`.
   - `h.tick` calls `d.tick`, which calls `sleepWatch.Observe` and records the sleep in the state record (`loop.go:544-556`).
   - `runNotices` reads `current.Sleeps` from that record, so the row exercises the sleep path.
   - If `sleeps` were dropped from the `AwakeSince` call, a 3h jump would exceed the grace and the row would fail.

4. **Paused intervals do not count toward the grace** (`pause.Intervals` passed to `AwakeSince`): **none.**
   - No row covers a pause interval inside the grace; `ACoveredChildSendsNothing` tests only `Covered`.
   - If `pause.Intervals` were dropped from the call, every test would still pass.
   - The summary and `doc.go` both claim this behaviour.

5. **The grace returns an empty finding and empty `activityStamp`**, so no agent-based notice goes and the episode keys are not cleared by `known`.
   - Partly pinned: the no-notice side is covered by row 1.
   - There is no assertion on marker or memory state afterwards.
   - A key written by a previous episode would be kept, because `activity.known` is false. That matches the earlier dead-driver return, so there is no new behaviour to pin.

6. **`memory.Since` drives the grace** (restart, resume, old episode).
   - `TestRunNotice_OncePerEpisode` covers a new running episode after a halt: it sets `Modified = now` and advances the grace before the second notice.
   - That is an adequate pin for "grace from the episode's since".

7. **A held loom bootstrap lock counts as alive** (`childDriverLockHeld`).
   - Pinned by `TestChildDriverLockHeld` / `AHeldLoomBootstrapLockReadsAlive`.
   - Dropping `LoomBootstrapLock` from the lock list makes it fail.
   - It is not a duplicate: the driver-lock and run-lock rows hold different paths, and `NoLockHeldReadsNotAlive` is the negative case.
   - The `noticeDriverAlive` level is untested, but `childDriverLockHeld` is the changed unit and it is covered.

8. **`doc.go` and the `DriverAlive` doc wording:** static prose, exempt.

**Findings**

- **LOW, design, `internal/dobbyengine/runnotice_test.go` (`TestRunNotice_Conditions`, new rows near line 120).**
  - No row pins that paused intervals are excluded from the grace.
  - The `pause.Intervals` argument at `runnotice.go:251` is untested.
  - The summary and `doc.go` claim paused time does not count.
  - Fix: add a `TestRunNotice_Conditions` row with `f.pause.Intervals` covering the time past the grace (`Covered` false) and `alive=false`, expecting no notice.
  - Evidence: `runnotice.go:251`; `passes_test.go:185-192` (`tick`); no existing row sets `pause.Intervals`.

**Tool calls**
1. Grepped for `suspended`, `Sleeps` and `SleepWatch` in the dobbyengine tests, to see whether `suspend` feeds recorded sleeps; the manifest lacked `tick`'s sleep recording.
2. Read `newNoticeHarness`, to confirm `Modified = clock.now` and the default `alive` / `pause`.
3. Grepped `Sleeps` in non-test dobbyengine files, to find where sleeps are recorded.
4. Read `loop.go:535-548` and grepped `sleepWatch`, to confirm `tick` observes the sleep and writes it to the state record.
