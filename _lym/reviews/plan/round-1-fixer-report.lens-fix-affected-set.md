```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun F1's search for the overview lines the new `internal/reportconfig` package forces, and check that the fix covers every one of them.

List of findings left unchecked: none. F1 was the only finding, and I checked it.

No findings.

F1's set is the `docs/overview.md` lines that name `internal/reportconfig`, `report.yaml` or `cost_min_runs`. My search found three:
- `:319` is the tree line. The fix's added sentence covers it.
- `:480` is the report module line's package list. The fix's added sentence covers it.
- `:483` names `report.yaml`'s `cost_min_runs` in the cost-report sentence. Card 27 covers it: "the report module's line names the `report` config module and its `cost_min_runs` key" (`_lym/plan/27-raddle-docs.md:9`).

So every member has a card. One note outside my lens: card 08 now says the report module line names `internal/reportconfig`, and card 27 still says the same line names "the `report` config module". The two cards overlap on that point but do not contradict each other. The consistency lens should judge it.

Tool calls:
- One grep of `docs/overview.md` for `reportconfig`, `report.yaml` and `cost_min_runs`. My context held only the parent's grep for "report", which was capped at 40 lines.
