# creelengine: journal a hold renewal as renewed, out of the log

`lyx creel log` showed a `held` event for every hold refresh.
A watcher renewing its hold every few seconds buried the posted, read and injected events under hundreds of `held` lines.
Now a hold's start logs one `held` event, its release logs one `released`, and a renewal by the same owner logs nothing.

Closes #722

## Design

- `Store.Hold` writes a `held` record when it starts a hold and a `renewed` record when the same owner renews a live one.
- The fold applies a `renewed` record to the hold exactly as a `held` one (owner and expiry) but returns before adding a log entry.
- The renewal is still journaled rather than dropped.
  The journal is the daemon's only durable state, so a restart must see the renewed expiry, or the hold would revert to its first expiry and let deliveries through mid-cycle.
- A hold taken after the earlier one expired or was released journals `held` again, so each start shows in the log.
- The `creel hold` CLI output is unchanged and still reports `renewed`.
  The orch watcher's renewal cadence is untouched.

## Compatibility

An older binary folding a journal with `renewed` records ignores the unknown event and logs it at Debug, per the tolerant-records invariant, so the load never fails.
Such a binary keeps the hold's first expiry, so the hold can lapse early under it.
Only one daemon runs per hub and a hold lasts at most an hour.

Each renewal still adds one journal line, which is acceptable since the change targets the log view.

## Tests

- `Hold_OwnerRenewsReleasesAndRetakesAfterExpiry` (renamed from `Hold_OwnerRenewsAndReleases`) pins the log as `held`, `released`, `held`, `held` across a renewal, a release, and a hold retaken after expiry.
  It reopens the store after the renewal and checks that another sender's hold is refused naming the renewed expiry.
- `TestJournal_Retention` gains a renewal inside the retention window and one carried across a day rollover, checked after a fold and after a reload.

The package doc's hold section describes the new log behavior.

## Not run by hand

`lyx gate test ./internal/creelengine` is the check for this change; nothing was run against a live daemon with a watcher holding a strand.
