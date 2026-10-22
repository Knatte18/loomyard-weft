# Round 2 fixer report

## Changes

- **F1 (BLOCKING), fixed as proposed**, in `_lyx/plan/04-board-owner-writes.md`.
  The premise holds: `retryBoltPush` (`internal/fabricengine/pushclassified.go:542`) runs only after a rejected first push.
  It defers only when its `lock.TryAcquireWriteLock` on `BoardWriteLockFile` fails, and it takes the push lock with the blocking `lock.AcquireWriteLock`.
  `PullThenCommitWritten` pulls under the board write lock immediately before the commit, so nothing in the old inputs rejected the first push.
  - The Intent's `OwnerRequest` sentence now names `BeforePush func()`.
    It runs once between the setter's commit and its first push, it is nil in production, and it is the test's only way to race a second clone.
  - Replay-loss row: `BeforePush` pushes a different `owner_machine` for the slug from a second clone.
    The first push is rejected, the retry replays, origin's value stands, and the post-push re-read reports the slug in `Lost` with origin's owner.
  - Deferred-push row: `BeforePush` pushes a commit touching another entry from a second clone and takes the board write lock, so `retryBoltPush` defers.
    A `Sleep` stub releases the lock on its first call for the retried-and-lands case, which reports the slug in `Recorded` with its owner on origin.
    It keeps the lock for the still-ahead case, which refuses naming `lyx board sync`.
  - The words "a push lock held across the retry" are gone.
    No target was added, so no `validate-plan --overlay` run was needed.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings, and no finding was left unchecked.
- fix-proposal, LOW: "a conflicting commit" in the deferred-push row could be read as a competing owner write, which would turn the lands case into a copy of the replay-loss row.
  Fixed as proposed: the row now pushes "a commit touching another entry", and the lands case asserts the slug in `Recorded`.

## Disputed

None.

## Deferred

None.
