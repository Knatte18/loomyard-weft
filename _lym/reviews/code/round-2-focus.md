---
round: 2
focus:
  - "internal/lock/tracked.go `refuseTrackedTree` and `TestAcquire_LockInTrackedLymDir` in internal/lock/tracked_test.go: does every acquirer refuse a lock path built beside a file under the Bolt checkout, as the Goal's check requires, and is the wording of pattern/PATTERN-tracked-file-locks.md and its PATTERN.md line true to what the test refuses; does any Bolt directory-name constant respect PATTERN-leaf-packages?"
  - "PATTERN.md lines for PATTERN-cwd-resolution and PATTERN-durable-vs-ephemeral-state: compared with their text before the headroom commit 4406aef5, does the diff drop or weaken any clause in an entry the Goal does not touch?"
  - "internal/shedadapters/settlerecord.go lock sites using `lock.TrackedFileLockPath` (settle record, settle patch, generation record, CarryRecords copy): does a test fail if one reverts to path plus .lock under a `_lym` directory?"
  - "internal/lock/tracked.go `refuseTrackedTree` loop: how does it treat a `.lym` segment that precedes a `_lym` segment, and does a test pin that choice?"
  - "internal/fabricengine/spawn.go doc comment on the rebase-free push, and the rows covering `TrackedLockPath` symlink roots, `TrackedFileLockPath` relative paths, gitrepo `PushLockPath` and internal/state lock directory creation: does each comment and behavior match the code as the diff leaves it?"
---
