```yaml
lens: fix-consistency
outcome: reported
```

Done, using one extra tool call. One finding, MEDIUM: a sentence card 6 already had now contradicts the bullet this round added.

**Finding 1 (MEDIUM)**
- **Changed text:** `_lym/plan/06-tapestry-config-module.md`, the new readers-list bullet: "`TestMigrateConfig`'s … subtest drops its `PlanLenses == "twill-plan"` conditions …, since `LoadConfig` leaves the plan fields at their zero value and only warns on the migrated `plan_lenses` key".
- **Problem:** The same card's Tests paragraph, which this round did not touch, still says "`TestLoadConfig_Loads` asserts the plan fields of `Config` come from `tapestry.yaml`". The new bullet, and the Intent's "`LoadConfig` never reads `tapestry.yaml`", both say `LoadConfig` leaves those fields at zero. A reader of card 6 now gets two opposite instructions for what `LoadConfig` yields for the plan fields.
- **Evidence:** `TestLoadConfig_Loads` reads plan fields after `LoadConfig` at `internal/loomengine/config_test.go:174-389`, for example lines 174-175, 212-214 and 285-286. The fixer's own search listed these lines.
- **Fix:** Rewrite the `TestLoadConfig_Loads` sentence the same way the `TestMigrateConfig` subtest was handled. Its plan-field assertions either drop out, or move to a config that has been through `LoadTapestryConfig` and `Config.WithTapestry`.

Twins I checked that agree:
- **Card 5 and card 6, `TestMigrateConfig`:** card 5 drops shed-override rows and card 6 drops the `PlanLenses` conditions of a different subtest, so the edits don't overlap.
- **Card 9 numbers:** Discussion-Write 3→2, Friction-Reflect 16→15, `6.6 Plan-Write`→`6.5`, `6.7 Plan-Burler-Review`→`6.6` all match `TestEntryNumbers` and `overview_test.go`.
- **Card 10 numbers:** after card 9, Code-Bouncer is 10 and Drift-Verify 11. Card 10's sentence correctly swaps them to Drift-Verify 10 and the Code entries 11.
- **Card 11, darn map:** Drift-Baseline 2→3 and Code-Bouncer 6→7 match.
- **Card 11, checklist rows:** `[ ] 2. Record-Write` and the rows after it through `[ ] 11. Friction-Reflect` match `checklist_test.go:488-495`. The `[x] 5. Darn-Write` and `[&gt;] 7. Code-Review` rows in that range shift by one too, and the card's "through" phrasing covers them.
- **Other cards:** no other card or `00-overview.md` names `TestEntryNumbers`, `TestRenderChecklist`, `TestDobbyLines` or `TestCheckResolved`, so there are no stale twins elsewhere.

Tool calls:
1. Grepped the other cards and `00-overview.md` for twin mentions, and printed `checklist_test.go:486-496`. My context lacked the other cards' text and the exact darn checklist lines.
