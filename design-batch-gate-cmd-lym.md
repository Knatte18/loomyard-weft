# Run the cmd/lym tests in the Webster batch gate so verify failures surface in the fork

`batch-gate-cmd-lym` · Task · enhancement, webster, gate · running · Describe

## Why (operator, 2026-11-08)

`docs/research/webster/webster-time-2026-11-08.md` found that 8 of 14 Webster runs failed the plan-level verify on its first attempt, with `cmd/lym` among the failing ids in 5 of those 8.
Each such run paid a verify-fix fork plus a second verify, about 6.8 min per affected run and 3.9 min per run on average.
A batch gate tests only the packages its cards touch, so a change that breaks a `cmd/lym` test is first seen at the plan-level verify, after the fork that made it has ended.

## What

- Make the Webster batch gate also run the `cmd/lym` tests (the verify tiers the plan-level verify runs for `cmd/lym`), so the batch's own fork sees and fixes such a failure at once.
- Read how the batch gate's scope is computed in `internal/websterengine` and the gate, and measure from the hub's gate run records (`.lym/gate/runs/*.json`) what a `cmd/lym` run costs.
  From that, choose between adding it to every batch gate and only to the last batch's gate, whichever saves more over the 14 runs in the report, and state the choice and its numbers in the change description.
- Keep this general, not loomyard-specific: a hub names the extra package set through the gate or language config default for Go hubs, with no hub step needed for loomyard (the loomyard default holds `cmd/lym`).
- The plan-level verify stays as it is.
