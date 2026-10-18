# creel: journal a hold renewal as `renewed`, kept out of the log

Fixes #722.

The orch watcher renews its creel hold on the orch strand on every tick of a context cycle.
Each renewal went through `Store.Hold`, which wrote a fresh `held` record, and the fold turned every `held` record into a log event.
`lyx creel log --to ly:orch` then showed a `held` event every couple of seconds, burying the posted, read and injected events.

`Store.Hold` now writes `held` only when it starts a hold, and a new `renewed` record when the same owner renews a live one.
The fold applies a `renewed` record to the hold like a `held` one, owner and expiry, but adds no log event for it.
A hold, any number of renewals and a release now log `held` and `released` only.

## Design decisions

- The renewal stays in the journal rather than in memory only: the journal is the daemon's only durable state, and without the record a daemon restart would revert the hold to its first expiry and could let deliveries through mid-cycle.
- A distinct event name, rather than a flag on `held`, lets the fold tell the two apart by event alone.
  An older binary folding a journal with `renewed` records ignores them at Debug, per the tolerant-records rule, and keeps the hold's first expiry.
- A hold taken again after an expiry or a release is a new `held` event, so each start stays visible.
- `creel hold` still answers `renewed: true` for a renewal; only the log view changes.

## Tests

`Hold_OwnerRenewsAndReleases` now reopens the store over the journal after the renewal, checks that another sender's hold is refused naming the renewed expiry, and expects the log to read `held`, `released`.
The package doc's Holds section states the new log behavior.
