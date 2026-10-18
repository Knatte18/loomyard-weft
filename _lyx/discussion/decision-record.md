# Decision record — creel-hold-log-spam

## Goal

`lyx creel log` shows one `held` event when a hold starts, not one per renewal, so a watcher renewing its hold every tick no longer buries the posted, read and injected events.

## Scope

- `internal/creelengine`: how `Store.Hold` journals a renewal by the same owner, and how the fold applies and logs it.
- The package doc's Holds and Log sections, for the new behavior.
- The existing hold renewal test's expected log events.
- Out of scope: the orch watcher's renewal cadence and `cycleHoldDuration` in `internal/orchengine`, and the `creel hold` CLI output, which still reports `renewed`.

## Decisions

### A renewal is journaled as its own `renewed` record

Decision: `Store.Hold` writes a `held` record when it starts a hold, and a `renewed` record when the same owner renews a live one.
The fold applies a `renewed` record to the hold exactly as a `held` one, owner and expiry, but adds no log event for it.
Rationale: the journal is the daemon's only durable state, so a renewal's later expiry must still be written, or a daemon restart would revert the hold to its first expiry and let deliveries through mid-cycle.
Keeping the renewal out of the log is what the issue asks for, and a distinct event name lets the fold tell the two apart without a new record field.

### A hold taken over after expiry or release is a new `held` event

Decision: a hold whose earlier one expired or was released journals `held` again, so the log shows each start, and `released` still marks each end.
Rationale: those are the changes the issue wants visible.

## Constraints

- `PATTERN-tolerant-records`: an older binary folding a journal with `renewed` records ignores the unknown event, logging it at Debug, and never refuses the load.
- `PATTERN-creel-store`: only `internal/creelengine` writes the journal; the change stays inside it.
- `PATTERN-test-economy`: the renewal behavior extends the existing `Hold_OwnerRenewsAndReleases` row rather than adding a test.

## Auto-mode assumptions

- "One when it ends or changes" in the issue means a release, an expiry or a new owner, not an extended expiry; the renewed expiry stays readable through `creel hold`'s answer.
- Journal growth from one line per renewal is acceptable, since holds last minutes and the issue is about the log view.

## Open risks

- An older binary reading a journal with `renewed` records keeps the hold's first expiry, so under it the hold can lapse early; only one daemon runs per hub, and a hold lasts at most an hour.

## Acceptance criteria

- Hold, renew and release by one owner log `held`, `released` and nothing for the renewal.
- After a renewal, a store reopened over the same journal holds the renewed expiry: another sender's hold is refused naming that expiry.
- The package tests pass under `lyx gate test ./internal/creelengine`.
