```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change the amended cards (07, 08, 23, 25) make, find what else reads or asserts it, and which cards cover that.

**Per-change list**

- **Card 7: new hub-wide `report` config module, now in `internal/reportconfig`.**
  - Set found:
    - `internal/configreg#Modules` (`configreg.go:170`)
    - `internal/configreg#TestModules_SeedOnlyAndHubWideFlags` (`configreg_test.go:58`)
    - `internal/configreg#TestModules_EveryTemplateKeyClassified`, which reads the template through the registry
    - `cmd/lym#configStrictnessDegradingSet` (`configstrictness_test.go:64`)
    - `docs/overview.md:319`, the package tree line
    - `docs/overview.md:480,483`, the report module line
    - `internal/reportcli/doc.go:26`
    - `pattern/PATTERN-config-layers.md:63`, the "Degrading callers" list
  - Covered by:
    - Card 7: `Modules`, `TestModules_SeedOnlyAndHubWideFlags` and `configStrictnessDegradingSet` as Edits; `TestModules_EveryTemplateKeyClassified` as a Use.
    - Card 8: `docs/overview.md` (its Intent names the report module line and the tree line) and `reportcli/doc.go`.
  - Not covered by any card: `pattern/PATTERN-config-layers.md:63`.
- **Card 8: `reportcli` reads `cost_min_runs` through `reportconfig.LoadConfig`.**
  - Set found:
    - `internal/reportcli/cost.go:59`, the call site
    - `internal/reportcli/doc.go:26`
    - `docs/overview.md:483`
  - Covered by: card 8 (`costCommand`, `reportcli/doc.go`, `//docs/overview.md`). Fully covered.
- **Cards 23 and 25:** the amendment changes nothing in them that reaches the report config module. No baseline exists to name any other change, so this lens found no new affected set to check.

**Findings**

1. **LOW, consistency, `_lym/plan/07-report-config.md` (Edit list and Intent).**
   - Summary: card 7 adds a new `LoadOrTemplate` caller, `internal/reportconfig`, but does not list `pattern/PATTERN-config-layers.md`. That file's "Degrading callers" set names its sibling `hibernateconfig` but not `reportconfig`, so the change makes the background's list wrong.
   - Evidence:
     - `pattern/PATTERN-config-layers.md:63`: the list includes `hibernateconfig` and has no `reportconfig`.
     - `internal/reportconfig/doc.go:8`: "LoadConfig degrades to the template".
     - `cmd/lym/configstrictness_test.go:64`: `"internal/reportconfig": true` in the degrading set.
     - `git show --stat 5ec85b9c`: no `pattern/` file was touched.
   - Mitigating: the list was already out of date before this card, since it lacks `sheddriver`, which also calls `LoadOrTemplate`. No test pins the background list.
   - Proposed fix: card 7 lists `pattern/PATTERN-config-layers.md` under `**Edit:**`, and its Intent adds `reportconfig` to the "Degrading callers" set. Card 7 sits in failed batch 7, so it can still be edited.

**Tool calls**

1. Grep for files that name `hibernateconfig` and `reportconfig`, to find the docs and lists that track sibling config packages. The manifest held no doc-side readers of the module.
2. Grep of `docs/overview.md`, the pattern background, the `configreg` and `reportengine` docs, and every reader of `cost_min_runs` or `report.yaml`. This confirms the overview lines and the list that omits `reportconfig`.
3. `sed` of `PATTERN-config-layers.md` lines 55–70, the production `LoadOrTemplate` callers, and `git show --stat 5ec85b9c`. This checks whether the list is meant to be complete and what card 7's landed commit touched.
