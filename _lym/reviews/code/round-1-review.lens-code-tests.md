```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Per behaviour, the pinning test**

All rows named below are in `TestWait_HeldTurnEndNotice` (`hold_test.go`).

1. An operator-typed turn start (`START:<text>`) not in the sent-texts file, then a held turn end, sends no notice, leaves `NotifiedOffset` at 0 and holds to the deadline.
   - Pinned by the row "a held turn end of a turn the operator typed sends no notice and leaves the offset" (`hold_test.go:336-343`).
   - It asserts no notices, offset 0 and `OutcomeTimeout`. If `operatorStartedTurn` or the new `wait.go:319` case broke, notices would be non-empty.
2. A turn start whose text the sent-texts file holds is notified.
   - Pinned by the "a parent send" turn in the second new row (`hold_test.go:352`, `sentTexts: ["a parent send"]`). The expected `«still missing»` fails if send-texts matching broke.
3. The run's prompt turn is notified, and so is a turn start with no instruction.
   - Pinned by the same row: `START\nSTOP:prompt turn` expects `«prompt turn»`, and the bare `START` turn expects `«notification turn»`.
4. After an operator-started held end, the held end of a later lym-started turn is notified, so `NotifiedOffset` was left alone.
   - Pinned by the second row, whose ordering is operator turn, then no-instruction turn, then sent turn.
   - If the skip had advanced `NotifiedOffset`, the later notices would be dropped by `notifyHeld`'s `held.offset <= NotifiedOffset` check. Row 1 asserts offset 0 directly.
5. The newest-turn-start-before-the-turn-end choice (D2).
   - Pinned by the second row. Each held end follows a different turn start, so picking the oldest or the first operator start would suppress `«notification turn»` or `«still missing»`.
   - The `>=` boundary cannot be hit, because a turn start's end offset is always below the turn end's. Nothing to pin.
6. Carry-over reads the same instructions as before (D1).
   - Pinned by the existing `operatorInstructions` and carry-over tests in `carryover_test.go`. The diff's one-line change adds `Raw` to `instructionEngine`, which `positionedTurnStarts` now needs, so those tests exercise the refactored path.
   - A behaviour change here is that a turn start whose `Raw` is empty or not found in the data is now dropped, where the old code kept it. No test pins that, but the claude engine always sets `Raw` (`claudeengine/signals.go`), so I am not raising it.
7. An engine without `SessionSignalParser`, or with no turn start before the turn end, is notified as today.
   - Pinned by the pre-existing rows in the same table. They use non-session engines and `STOP`-only events and still expect a notice (`hold_test.go:105`).
8. Doc paragraph in `doc.go`: static prose, exempt.

**Findings**

- LOW, scope. Location `hold_test.go` (the new rows, around lines 336-363), evidence `wait.go:319`, `hold.go:91-99`.
  - Summary: the new skip case is not covered on a `Spec.HoldDeferral` run.
  - On such a run the skip also stops `run.pendingHold` from being queued by the `case !run.turnStartRead` arm at `wait.go:327`. No row has an operator turn with `HoldDeferral` set, so a regression that queued and later sent a deferred notice for an operator turn would pass.
  - Fix: add a `holdDeferral` row with an operator-typed turn, expecting no notice after `holdNoticeSettle` and `holdNoticeMaxDeferral`.

**Not findings**

- The new fake `START:<text>` support in `fakes_test.go` has no direct test, but the two new rows use it.
- Test economy: the new rows extend the existing tier0 table, as `PATTERN-test-speed` requires, and no new test function duplicates coverage.

**Tool calls:** none. My inherited context already held the diff, `hold_test.go` and `carryover_test.go` excerpts, and `signals.go`.
