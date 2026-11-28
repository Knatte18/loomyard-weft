# The last batch gate covers cmd/lym and cross-package tests, so the plan-level verify seldom fails

`batch-gate-covers-cmd-lym` · Note · enhancement, gate, webster, weaver

## Seen (`docs/research/webster/webster-time-2026-11-08.md`, lever 3)

8 of 14 rows failed the plan-level verify on attempt 1, with `cmd/lym` in the failing ids of 5,
and each paid a verify-fix session plus a second verify: 3.9 min per row, 6.8 per affected row.

## Idea

Add `cmd/lym` to the batch gate's scope, or to the last batch's gate, so a batch session that can fix the failure meets it at once.
The cost of the added gate run is measured before this is built.
