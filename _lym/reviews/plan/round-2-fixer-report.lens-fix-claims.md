```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims added by the card 13 fix hold against the code at HEAD.

**[LOW]** `_lym/plan/13-gate-mutate.md`, the new lines under the budget rows: "`redundancy_build_timeout_min: 100000`, far above the real coverage build time"
Summary: whether 100000 is far above the real build time depends on the row's `minuteUnit`, and the card never names that unit.
Evidence: the build bound is `RedundancyBuildTimeoutMin * minuteUnit` (`internal/gatecli/redundant.go:255`, `internal/gatecli/redundancy.go:462`). The row's unit is described only as "so short that the budget is spent at the hold". With a 1µs unit the bound is 100ms, which is below a real `go` coverage build. With a 1ms unit it is 100s, which is enough. So the claim holds only if the unit is about a millisecond or longer, and the card doesn't say so. An implementer could pick a nanosecond-scale unit, and the build would time out again. Fix: name the unit (for example 1ms), or state the requirement as a product, e.g. the bound times `minuteUnit` is well over a minute.

The other claims check out:
- "The coverage build's bound and the budget share the one `minuteUnit`" matches the code. `redundantCmd` takes a single `minuteUnit` and scales the build bound with it (`redundant.go:105`, `:255`). The redundancy step scales its build bound with `step.minuteUnit` as well. Card 13's `mutateCmd` signature has one `minuteUnit`.
- `mutation_budget_min` is a real key. Card 4 adds it (`_lym/plan/04-gate-config-reach.md:25`, `:31`), so setting it in the row's hub config is valid.

Tool calls:
1. grep of `minuteUnit` and `mutation_budget_min` across `internal/gatecli/redundant.go`, the redundant integration test, `internal/gateslot` and `_lym/plan`. My context didn't confirm that the budget key exists in the plan or how `redundantCmd` uses its unit.
