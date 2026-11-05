```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens over the amendment's changed text (cards 07, 08, 23, 25).

**Per guard, reachable or not**

- Card 07, `LoadConfig` refusing a `cost_min_runs` under 1 and naming the key and `lym config report`: **reachable.**
  The input is a hub `report.yaml` holding `cost_min_runs: 0`, read through `configengine.LoadOrTemplate`.
  No upstream check rejects it on that path: `ConfigSpec` only sets a category for the key and checks no value (`internal/reportconfig/config.go:35-44`).
  `configreg`'s `CheckLoad: checkLoad(reportconfig.LoadConfig)` runs this same function, so it is the same guard at a second entry, not a separate upstream rejection.
  A hand-edited hub file skips the `lym config --set` path and still reaches the guard at `lym report cost`, through `reportcli/cost.go:59`.
  The test row `ZeroRefused` reaches it (`internal/reportconfig/config_test.go:29`).
  The card names this row in its Intent: "a zero value refused naming the key" (`_lym/plan/07-report-config.md:33`).
  The guarded site does not recur unguarded: `reportconfig.LoadConfig` is the one reader of the key (`internal/reportcli/cost.go:59`).
- Card 08: the amendment changes only which `LoadConfig` it uses (`reportengine` to `reportconfig`). That adds no guard, refusal, branch or test row. Its bad-flag refusals are settled text outside the amendment.
- Card 23: the amendment moves doc-file targets from `Extends` to `Edit` and adds doc-wording sentences. It adds no guard, refusal, branch or test row.
- Card 25: the amendment moves `pawlengine/doc.go` from `Extends` to `Edit` and adds a doc sentence. It adds no guard, refusal, branch or test row.

**Findings:** none.

**Tool calls:** none. Every claim above rests on files already read in the inherited context.
