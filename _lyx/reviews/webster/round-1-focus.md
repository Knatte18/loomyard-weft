---
round: 1
focus:
  - "Cards 2, 3, 12 (`shedengine.Relaunch`, the `--arbiter-attempt` flag, the consumption hook): settle whether consumption happens under the status lock before the loop reads files or spawns, checks the attempt against the stop key (`current_producer`, history length, `state`), marks `applied`, and performs the goto while keeping the pause, and whether a refused or repeated attempt returns the error envelope carrying `arbiter_attempt`."
  - "Cards 9 and 10 (eligibility, arbiter verb): settle whether every failure path (budget exhaustion, timeout, invalid or missing verdict, ineligible stop, spawn error) falls back to hand-on and never loses a stop, and whether `attempt.json` is written before the spawn so it counts against `arbiter_max_spawns`."
  - "Cards 14, 15, 16 (driver stencil branch, batten hold, revive): settle whether the batten hold releases when the marker clears or goes stale, and whether the strand revive can never relaunch an arbiter strand."
  - "Cards 1, 4, 5, 11, 13: settle whether the arbiter records, paths and config keys match the plan's Shared Decisions (`records-in-loomengine`, `attempt-identity`), whether `internal/battencli` imports no other CLI package, and whether `--by arbiter` admits only the run's own arbiter."
  - "Per-card mechanical check and test economy across the range: settle whether each card's groups had their type-specific checks run against every target, and whether new tests duplicate coverage by an existing public-surface test."
---

Seed focus drawn from the plan's cards; no round has been judged yet.
