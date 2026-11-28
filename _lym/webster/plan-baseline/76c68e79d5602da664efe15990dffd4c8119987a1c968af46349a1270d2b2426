# Card 6 — push-lock-holder-core

**Create:**
- `plan:internal/fabriccore#PushLockHolder` -> `type PushLockHolder struct`
- `plan:internal/fabriccore#DescribePushLockHolder` -> `func DescribePushLockHolder(holder PushLockHolder, found bool, readErr error, now time.Time, alive bool) string`
- `plan:internal/fabriccore#SidesProgressed` -> `func SidesProgressed(before, after map[string]string) bool`
- `plan:internal/fabriccore_test#TestDescribePushLockHolder` -> `func TestDescribePushLockHolder(t *testing.T)`
- `plan:internal/fabriccore_test#TestSidesProgressed` -> `func TestSidesProgressed(t *testing.T)`
- `plan:internal/fabriccore#FlaggedLoopNext` -> `func FlaggedLoopNext(stepRan, acquired, ownedFlagSet bool) (again, reportAcquired bool)`
- `plan:internal/fabriccore_test#TestFlaggedLoopNext` -> `func TestFlaggedLoopNext(t *testing.T)`

**Edit:**
- `internal/fabricengine#describePushLockHolder`
- `internal/fabricengine#holdPushLock`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine#coalesceFlagged`
- `internal/fabricengine#TestDescribePushLockHolder`

**Delete:**
- `internal/fabricengine#pushLockHolder`

**Extends:**
- `internal/fabriccore/doc.go`

**Uses:**
- `internal/fabricengine#TestCoalesceFlagged_HolderRecordLivesWhileTheLockIsHeld`
- `internal/fabricengine#TestCoalesceFlagged_PendingFlags`

**Intent:**
The push lock holder's busy clause and the coalescing loop's progress test become core rules; the record file, the pid probe and the lock stay in the engine.

- `PushLockHolder` is today's `pushLockHolder`, `PID` and `Since`, with the same JSON tags, so the holder file's bytes are unchanged.
  `holdPushLock` writes it.
- `DescribePushLockHolder` returns today's four clauses:
  no record (found false);
  unreadable, naming `readErr`, or a record with no pid or no time;
  a pid no running process has, with its age, as possibly stale (`alive` false);
  and held by the pid for its age.
  The age is `now - Since` rounded to the second.
- The engine's `describePushLockHolder` reads and decodes the file, asks `alive(pid)` only for a decoded record with a pid, and returns the core's clause, so `pushLockBusyError`'s text is unchanged.
- `SidesProgressed` reports whether any side's HEAD in `after` differs from `before`, a missing side reading as "".
  `coalescePushSides`' step collects both sides' HEADs before and after and returns it, as today's comparison does.
- `FlaggedLoopNext` is `coalesceFlagged`'s loop rule (`pendingflags` of the record's Goal):
  a missed acquire before any step ran reports not acquired and leaves the flags to the holder;
  a missed re-acquire after a step ran reports acquired;
  after a released step, an owed flag set again sends the loop back to acquiring, and none ends it.
  The engine keeps the flag files, their atomic writes, the rename-take and the lock.

Tests:
- `TestDescribePushLockHolder` (core, tier0) takes the engine test's rows `LiveHolder`, `DeadHolder`, `Corrupt`, `EmptyObject` and `NoRecord`, with fixed times.
  The engine's `TestDescribePushLockHolder` keeps its `NoRecord` and `Corrupt` rows, which pin the file read's mapping to found false and to `readErr`,
  and deletes `LiveHolder`, `DeadHolder` and `EmptyObject`, held by the core rows.
- `TestSidesProgressed` (core, tier0): no side moved; one side moved; a side that turned unborn; an empty map.
- `TestFlaggedLoopNext` (core, tier0) walks each branch of the loop rule.
  `TestCoalesceFlagged_PendingFlags` keeps its rows, which pin the flags on disk the rule reads.
The three core tests add under 1 second to one tier0 run.

**ImpactSummary:** Two private engine readers delegate to the core; the holder file format and every busy message are unchanged; no caller outside the files changes.
