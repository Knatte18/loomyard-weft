---
round: 1
focus:
  - "internal/gatecli/tiers.go and tiers_test.go: does `--upto N` select exactly the declared tiers 0..N in ascending order, skip manual tiers, drop covered tiers, and combine with `--tier` as the entry asks? Check gaps, N above the highest tier, and a manual tier named by `--tier`."
  - "internal/gateslot/tiers.go and config_test.go: is `manual` parsed and validated at load, and is `manual: true` on tier0 refused without breaking the no-tier-flag path?"
  - "internal/gatecli/status.go and internal/gateslot/pool.go: is `lyx gate status` read-only (no lock taken, no file created, no record written), and does it report taken slots with run, tier and start time as the entry asks? Check `Pool.Holders` slot filling and `Pool.Draining` against stale records and a pool with no drain lock."
  - "Scope: the diff adds a negative-`--upto` refusal, a `draining` field, a stderr note for skipped manual tiers and a CLAUDE.md push-gate edit. Settle which the amended board entry licenses."
  - "Claims in _lyx/landing/summary.md against the diff, notably that `--upto 2` runs the same tiers as the old push gate, and that docs (gatecli and gateslot `doc.go`, `docs/overview.md`, template.yaml) match the behavior."
  - "Test economy: new test rows against existing coverage in TestPlanTierRuns, TestGateTest_Scenario and the pool tests; behaviors added with no coverage."
---

Seed focus only; the reviewer still reads the whole committed diff.
