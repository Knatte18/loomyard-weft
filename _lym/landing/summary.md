# Retry a busy landing push and name the push lock's holder

A landing push that finds the records push lock held no longer halts the run after one 30s wait.
`pullRequestDepsGuardedBy` in `internal/loomcli/landingdeps.go` now retries the push after pauses of 30s, 1m and 2m, and halts only when the lock is still busy after the last one.
Another session's push holds the lock only while git runs, so the retries ride out that contention.
Any error that is not `ErrPushLockBusy` ends the retry at once.

When the lock is still busy, the error now says who holds it.
Every holder of `fabric.push.lock` writes a holder record, `fabric.push.holder`, beside the lock, with its pid and the time it took the lock.
The busy error reads the record and reports one of:

- `held by pid N for D`
- that the record names a pid with no running process, so it may be stale
- that the record is unreadable
- that no record exists

Closes #37

## Design decisions

- **The record sits at every holder site**: the coalescing push loop (`coalesceFlagged`), the origin reconcile's `tryRecordsLocks` on a weft, and the weft replay lock in `lockWeftForReplay`.
  Bolt's locks get no record, since the busy error reports the weft lock.
- **The release removes the record before it frees the lock**, so a holder never deletes the record of the next holder.
- **A record that cannot be written or removed is logged at Warn and the lock is still held and released.**
  The record only feeds a busy report.
- **No lock is cleared for a dead holder.**
  The push lock is an OS lock that its holder's exit frees, so a dead process cannot hold it; only the record can go stale, and the next holder overwrites it.
  The issue's "clear a lock whose holder is dead" has nothing to clear, so the diff reports a stale record as stale and does nothing more.
- `internal/fabricengine/doc.go` documents the record, and `cmd/lym/destructiveguard_test.go` allowlists the record's `os.Remove` in `pushlockholder.go`.

## To check by hand

- The issue asked to clear a lock whose holder is dead, and `internal/fabricengine/pushlockholder.go` does not do so, for the reason above.
  Confirm the omission is acceptable.
- The retry pauses in `landingPushRetryPauses` are a judgment call: about 3.5 minutes of waiting on top of the 30s per attempt.
  Check they suit real push contention.
- No regression check and no live run against a held lock were done; the tests cover the retry loop, the busy-report wording and the record's lifecycle at each lock site.
