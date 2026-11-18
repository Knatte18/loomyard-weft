```yaml
lens: fix-consistency
outcome: reported
```

I checked the card 13 budget-row edit against the sibling cards, rows and cross-references, and found nothing wrong.

- **Config keys exist:** the new text sets `mutation_budget_min: 1` and `redundancy_build_timeout_min: 100000`. Card 4 adds `mutation_budget_min` to `Config` and `configFile` (`_lym/plan/04-gate-config-reach.md:25,31`). `redundancy_build_timeout_min` already exists, and the config has no upper limit that would refuse 100000 (`internal/gateslot/config.go:41,293`).
- **Agrees with the Intent bullets:** the rationale matches card 13's own lines 81–82. The coverage build and planning run whatever the budget, each build is bounded by `redundancy_build_timeout_min`, and every mutant left past `mutation_budget_min` is reported unjudged.
- **No twin left unchanged:** the `reach_budget_min` row starts no build once its deadline is spent, as the review notes, so it needs no matching config line. No other card mentions the budget rows or how they are configured.
- **List structure holds:** the row bullet now ends with a period and continues on indented lines that end with a `;`, so the sub-list still reads correctly.

Further tool calls:
- grep of `_lym/plan` for `mutation_budget_min`, budget rows and `redundancy_build_timeout_min`: my context had no sibling cards, so I needed to find where they define or mention these.
- grep of `internal/gateslot/*.go` for `RedundancyBuildTimeoutMin`: I needed to check whether the config caps the 100000 value, which no file in my context showed.
