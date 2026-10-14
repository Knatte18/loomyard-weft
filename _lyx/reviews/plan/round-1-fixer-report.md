# Fixer report — plan round 1

## Changes

- **F1** (fixed as proposed): card 4's Firing paragraph now has the `Added` callback keep its `EntryRef` in a variable of `Watch` before `updateRequest`, and the `Pause`-error path lifts that kept ref through `Lift` when the request records no `Owned` entry, so the notice reads that its pause was lifted.
  It also states that the lift and notice run even when the `failed` write fails, and that `Watch` then returns that error, since the directory-replaced request file in the new sub-row makes that write fail too.
  The `TestWatch` `Pause`-error row gains the sub-row: an `onPause` hook replaces the request file with a directory, and the row asserts an error, the hub entry gone from the live list and a lifted-pause notice.
  To give the row its hook, card 3's `fakeWorld` description gains an `onPause` hook that the fake `Pause` runs before `req.Added`.
- **F2** (fixed as proposed): card 4's `--after-all` row gains a sub-row listing a slug that the missing-slug switch reports as not found, beside one `done` run, asserting it fires on the first reading and does not end `failed`.
- **F3** (fixed as proposed): card 7's Intent gains a bound sentence saying that the shortcuts run `lyx tender pause`/`resume` in the caller's own environment, so tender's caller check applies unchanged.
- **F4** (fixed as proposed): removed `internal/tendercli#callerOf` from card 3's Uses list, and the Intent's prose reference stays.
- **F5** (fixed as proposed): card 7's Intent adds the tender module bullet in `docs/overview.md` to the lines that change, now naming `lyx p` and `lyx r` beside `lyx pause` and `lyx resume`.

## Disputed

None.

## Deferred

None.
