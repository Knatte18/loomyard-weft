```yaml
lens: fix-proposal
outcome: reported
```

I checked F12, which the fixer fixed as the reviewer proposed. One finding.

**Unchecked list:** none.

**[MEDIUM] Whether 100000 is enough depends on `minuteUnit`, which the row never pins**
- **Changed text:** `_lym/plan/13-gate-mutate.md` Tests, in the `mutation_budget_min` row: "sets `mutation_budget_min: 1` and `redundancy_build_timeout_min: 100000`, far above the real coverage build time".
- **The case that breaks:** the build bound is `RedundancyBuildTimeoutMin * minuteUnit` (`internal/gatecli/redundant.go:255`).
  - The row only asks for a `minuteUnit` "so short that the budget is spent at the hold". A microsecond reads naturally as that.
  - With a microsecond, 100000 units is 100ms. A real Go coverage build takes seconds, so the build still times out and no mutant is planned. That is the failure F12 describes.
  - The claim "far above the real coverage build time" holds only when `minuteUnit` is at least about 1ms, which gives a 100s bound.
  - Config validation sets only a lower bound on the key (`internal/gateslot/config.go:288-301`, `want at least 1`), so nothing caps the value.
- **What to change:** pin the row's `minuteUnit` to a millisecond. The 1ms budget is then gone long before the build ends, and the bound is 100s. Or state the bound as a product, `redundancy_build_timeout_min × minuteUnit`, that must exceed the build time.

Tool calls:
1. Ran grep on `internal/gateslot` and `_lym/plan` to find where the budget keys are defined and whether the build-timeout key is validated. My context did not show the config code.
2. Read `internal/gateslot/config.go:280-305` to confirm the build-timeout key has no upper bound. My context lacked that span.
