---
round: 1
focus:
  - "Cards 3 and 5 in the closure `gate` body: after the pass-at-cap removal, check that no `logPassedAtCap` caller, `pass_on_cap` path or cap-related branch survives, and that the ping branch still runs only while a delivered round is pending."
  - "Card 3 keep-alive pings: check the per-round count is capped at three, is persisted so a restart cannot reset it, and that a ping never fires on a settled round or after a verdict."
  - "Cards 12, 13 and 14 wait loop: check the 5-second re-check throttle applies only to a pending gate with no new arrival, and that the off-boundary expiry probe is read-only, finalizes `done` at the bound, and cannot end a wait early."
  - "Cards 7, 8 and 9 round and approval edge cases: check that a fresh spawn reuses only a latest round holding no request, that `approve --review` refuses a missing, unreadable and empty file with distinct messages, and that `delivered` on a round with a verdict exits 0 and writes nothing."
  - "Card 6 and card 10 refusals: check the last-gate refusal covers every path that builds a gate list, and that each review refusal names `lyx batten status <slug>` for a slug and `lyx loom status` for none."
  - "Per-card mechanical checks and test economy: check each card's groups ran their type-specific checks against every target, and that new top-level tests stay within `cmd/lyx/testdata/test-budget.yaml` without duplicating an existing test's coverage."
---

Seed focus drawn from the plan's card index, covering the behavior-bearing cards first.
