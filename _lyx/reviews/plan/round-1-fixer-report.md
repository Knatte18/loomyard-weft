# Fixer report — plan round 1

## Changes

- F1 (LOW), fixed as proposed: card 2's Intent now says `stepLookupContract`'s doc comment gains a line covering the kept-key rows on `get`, `list-full` and the task envelope of `upsert`, `promote` and `merge`.
  The premise holds: the comment at `internal/boardcli/cli_test.go:573` names only the lookup contract on get, set-status, remove and set-deps.
- F2 (NIT), fixed as proposed: card 1's Intent no longer adds a `TestRunLock` row; it sets `unknown` on the existing accepted row "set-status to another value", whose act at `internal/boardengine/runlock_test.go:77` is the same `SetStatus("locked", strPtr("paused · X"))`.
- F3 (NIT), fixed as proposed: the set-status row's `saveOrder` in card 1 now starts with the `"status":` member the row writes, then `"zz_nested"` and `"zz_scalar"`.
  `Task`'s field order puts `status` after `body`, and the only later members (`short_name`, `created`, `revised`, `run_settings`) are absent on the seeded `a`, so the anchor pins kept keys after every known field.

## Disputed

None.

## Deferred

None.
