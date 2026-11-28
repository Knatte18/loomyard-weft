---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- **F1** (`internal/orchcli/status.go`): the `strand_down` help builds its cause list from the constants.
- **F2** (`internal/reedengine/apply.go`, `ownwindow.go`, `lifecycle.go`, `reconcile.go`): new cause `DownPanePlacementUnread` (`pane-placement-unread`) for a pane the session lacks when the server-wide lookup failed.
  `strandsElsewhereLocked` reports the failure as the set of panes the lookup was for, rather than a flag, so `Status` needs no second own-window test.
  Reconcile discards that set and plans as before.
  The help names the new cause, and the failed-lookup Warn no longer says the panes read as found nowhere.
- **F3** (`internal/reedengine/lifecycle_test.go`): added the `unbound` strand row.
  The failed-lookup loop also asserts every strand's `Down`, which pins F2.
- **F4** (`internal/orchroute/live.go`): both `Live` branches use `downReadingLogMessage`.

## Commits

- `6efb211be`: F1, F2, F3.
- `31f832e21`: F4.
- `659ee0b6d`: F4, formatting of the two changed calls, found by the self-check.
- `3e1fd0f25`: F2, repair of a gate finding.
  The `DownPanePlacementUnread` comment wrapped mid-sentence, and is now one sentence per line.

## Verification

`lym gate test` passed in tier0 on `internal/reedengine`, `internal/orchcli` and `internal/orchroute` after the first group,
on `internal/orchroute` again after each F4 commit's change,
and on `internal/reedengine` after the comment repair.
Tests of higher tiers were skipped in those runs.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, the two F4 log calls were not gofmt-formatted: fixed as proposed (`659ee0b6d`).

## Disputed

## Deferred

## Rulings
