# Card 1 — Bounded write-lock acquisition

**Create:**
- `internal/lock#AcquireWriteLockWithin`

**Edit:**
- `internal/lock/lock.go`
- `internal/lock/lock_test.go`

**Uses:**
- `internal/lock#AcquireWriteLock`
- `internal/lock#TryAcquireWriteLock`

**Intent:**
Add `AcquireWriteLockWithin`, an exclusive lock acquisition that waits at most `wait` for the lock.
It reports `(lock, true, nil)` once acquired and `(nil, false, nil)` when the lock is still held when `wait` runs out, so an expiry is not an error.
Implement it over gofrs/flock's `TryLockContext` with a `context.WithTimeout` of `wait` and a short fixed retry delay; a context expiry maps to `(nil, false, nil)`, any other flock error is returned wrapped.
The status-side push of card 3 needs it: a status push must give up on fabric's absorbing push lock after a bounded time, so a run is never stalled behind a hung detached push child.
The bound is the caller's `wait`, so a test shortens it rather than waiting out a production value.

Extend `lock_test.go` beside `TestTryAcquireWriteLock` with one table-driven test: a free lock is acquired within the wait; a lock held by another `AcquireWriteLock` holder returns `(nil, false, nil)` after a short wait; the same path is acquired once the holder releases.

**ImpactSummary:** additive: one new exported function in `internal/lock`; no existing caller changes.

**Commit:** 1: lock: bounded write-lock acquisition
