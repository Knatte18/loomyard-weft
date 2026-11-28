# Finalize gates a parent ahead by owed landings at its local head

With local landing, a parent branch can sit ahead of origin by landing tips whose push is still owed.
Finalize lands on top of those tips, but its lock-free merge and gate took the upstream tip as the gated tip.
Under the parent lock the parent's head was then always past the gated tip, so every such landing lost each of its `landing.finalize_lost_races` races.
It then merged and verified under the parent lock, and other landings on the same parent waited for that verify.

## What changes

- `parentMergeTip` in `internal/landingshed/parentalign.go` takes a `takeOwedLocal` flag.
  When it is set and the parent is ahead of its upstream, it asks `Deps.ParentAheadByOwed`.
  A parent ahead by owed landing tips alone is merged at its local tip, read under the parent lock as the push-skipped path already does.
  Any other ahead parent still merges the upstream tip, and the locked pass's sync refuses it as before.
- Finalize's lock-free pass sets the flag, so its gated tip is the head the locked pass compares with.
  A parent that another landing moves in between still loses the race as before.
- Parent-Align passes `false` and keeps merging the upstream tip, so it never merges a local landing that has not been pushed.
- A failed read of whether the parent is ahead by owed tips alone is an error from the lock-free pass, with the same wording the locked pass's sync uses.

## Design decisions

- The flag is per caller rather than a change for both producers, since Parent-Align's contract is to merge only what origin holds.
- The local tip is read under the parent lock, as in the existing local-tip path, so the read never sees a landing half made.

## Tests

- `TestFinalize_GatesWithNoLockAndLocksToLand` gains a row for a parent ahead by owed landing tips:
  it gates once with no lock, merges the local head, and lands with no verify under the lock.
  Before the fix the row merged the upstream tip and verified under the lock after its lost races.
- `TestFinalize_LockFreePassStops` gains a row for a failed owed-tips read, which ends the call before any merge.

## Merge danger

**Door:** two-way
**Blast radius:** landing
