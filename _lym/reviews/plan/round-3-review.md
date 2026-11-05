---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lym/plan/07-report-config.md
    summary: "card 7 still builds the report config module in internal/reportengine, which the amendment moves to internal/reportconfig to avoid an import cycle"
    fix: >-
      Rewrite card 7 so it creates the module in `internal/reportconfig`, as the decision record's Added after Discussion entry "The report config module lives in internal/reportconfig" says.
      Under **Create:** list `internal/reportconfig/doc.go`, `internal/reportconfig/testmain_test.go`, `internal/reportconfig/template.yaml`, and `plan:internal/reportconfig#Config`, `#ConfigTemplate`, `#ConfigSpec`, `#LoadConfig` and `#TestLoadConfig` with the signatures the card carries today, replacing every `internal/reportengine` target.
      Delete the `**Edit:**` line `internal/reportengine#allowedImports` and the Intent sentence "`allowedImports` gains `internal/configengine`", since `internal/reportengine` no longer imports `internal/configengine`.
      In the Intent, say the module sits in `internal/reportconfig` because `internal/reportengine` already depends on `internal/configreg`, and `configreg.Modules` gains `{Name: "report", Template: reportconfig.ConfigTemplate, HubWide: true, CheckLoad: checkLoad(reportconfig.LoadConfig), Spec: reportconfig.ConfigSpec}`.
      Add the Intent line "`testmain_test.go` is untagged, the package's `TestMain`.", which the `test-tier-unstated` check demands for a created `testmain_test.go`.
      Keep the rest of the Intent (the template line, `Config.CostMinRuns`, the refusal under 1 naming `lym config report`, the cross-pair `ConfigSpec`, `TestLoadConfig` rows).
      Checked with `lym loom validate-plan --overlay` against the baseline run of `{}`: the only new blocking findings are `create-already-exists` on card 7's new targets, because the landed tree already holds the package (commit 5ec85b9c);
      the baseline reports the same check on cards 8 to 12 for the same reason, and Webster's auto-rebaseline of a failed batch's cards clears it.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lym/plan/08-report-cost.md
    summary: "card 8 reads cost_min_runs through plan:internal/reportengine#LoadConfig, which the amendment replaces with internal/reportconfig LoadConfig"
    fix: >-
      In card 8's **Uses:** list replace `plan:internal/reportengine#LoadConfig` with `plan:internal/reportconfig#LoadConfig`.
      In the Intent's `costCommand` paragraph replace "it reads `cost_min_runs` with `reportengine.LoadConfig` into `MinRuns`" with "it reads `cost_min_runs` with `reportconfig.LoadConfig` into `MinRuns`".
      Nothing else in the card changes.
      Checked with `lym loom validate-plan --overlay` together with the card 7 rewrite of F1: no blocking finding beyond the baseline's `create-already-exists` pattern for landed cards.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/07-report-config.md
    summary: "card 7 edits configreg TestNames, which needs no change, and omits the two pinned sets a new hub-wide module that loads with LoadOrTemplate breaks"
    fix: >-
      Fold into the card 7 rewrite of F1.
      Under **Edit:** drop `internal/configreg#TestNames` and the Intent clause "and `TestNames` its name": `TestNames` in `internal/configreg/configreg_test.go` iterates `Names()` and pins ordering and non-empty templates, so a new module needs no row there.
      Add `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`, whose `wantHubWide` literal lists every hub-wide module by name and fails for a new `HubWide: true` module `report` until it names it.
      Add `cmd/lym#configStrictnessDegradingSet` (`cmd/lym/configstrictness_test.go`), the pinned set of packages that load config with `LoadOrTemplate`, which must gain `internal/reportconfig`.
      Say both in the Intent in one line each.
      Both glyphs were accepted by `lym loom validate-plan --overlay`.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/plan/23-raddle-recipe.md
    summary: "cards 23 and 25 list doc.go files under Extends although adding raddle or the hand-off marker rewrites an enumerating sentence, and most of those files get no stated change"
    fix: >-
      Card 23: move `internal/loomcli/doc.go`, `internal/shedrun/doc.go`, `internal/loomrecipe/doc.go`, `internal/shedcli/doc.go` and `internal/runprofile/doc.go` from **Extends:** to **Edit:**, leaving only `internal/shedrecipe/doc.go` under **Extends:** (its per-row sections take a new RaddleWrite and RaddleFile section without touching old text).
      Add one Intent sentence naming what each of the five gets: `shedrun/doc.go` line 5's list of recipe names gains `raddle`; `loomrecipe/doc.go`'s three recipe lists (the `New` and `Routing` line, the `Rows` and `RowNames` line, the `AmendmentTarget` line) gain it; `shedcli/doc.go` line 12's table of recipe names gains it; `runprofile/doc.go`'s recipe list and its defaults sentence (`solid` for tapestry and motif, `light` for darn and pawl) gain raddle with `light`; `loomcli/doc.go` rewords the sentence "A finished run of a recipe with no Finalize row, a motif run, makes its entry a Sketch" so it no longer covers a raddle run, then adds the Raddle-File sentence the card already describes.
      Card 25: move `internal/pawlengine/doc.go` from **Extends:** to **Edit:** and add the Intent sentence that its line listing the files under the beam's `.lym/beam/` (land request, reopen request, dobby-stop marker, await marker) gains the hand-off marker and its Beam-Up description gains the hand-off step.
      Checked with `lym loom validate-plan --overlay` applying both moves: no blocking finding beyond the baseline.
---

### [BLOCKING] Card 7 builds the report config module where the amendment forbids it

**Location:** `_lym/plan/07-report-config.md`

**Issue:** `_lym/loom/amendment.yaml` names the decision-record entry "The report config module lives in internal/reportconfig".
It moves card 7's targets from `internal/reportengine` to `internal/reportconfig`, because `internal/reportengine` depends on `internal/configreg` and `configreg.Modules` would have to import the module, a cycle Go forbids.
Card 7 still creates `internal/reportengine/template.yaml` and `reportengine#Config`, `#ConfigTemplate`, `#ConfigSpec`, `#LoadConfig` and `#TestLoadConfig`, and edits `reportengine#allowedImports`, so the entry's change is uncarried for this card.
Card 7 belongs to batch 7, which is failed, so Webster's auto-rebaseline accepts the edit.
The tree already holds `internal/reportconfig` (commit 5ec85b9c), so the plan is the stale side.

### [BLOCKING] Card 8 still reads the config through the reportengine module

**Location:** `_lym/plan/08-report-cost.md`

**Issue:** The same entry says card 8 uses `reportconfig#LoadConfig` in place of `reportengine#LoadConfig`.
Card 8 keeps `plan:internal/reportengine#LoadConfig` in **Uses:** and `reportengine.LoadConfig` in the `costCommand` paragraph.
Once card 7 is corrected as F1 says, that `plan:` ref resolves to nothing, so the plan gate refuses the card.
The tree's `internal/reportcli/cost.go` already calls `reportconfig.LoadConfig`.

### [MEDIUM] Card 7's edit list names a test that needs no change and misses two that break

**Location:** `_lym/plan/07-report-config.md`

**Issue:** `internal/configreg/configreg_test.go`'s `TestNames` walks `Names()` and asserts order, templates and loaders for whatever modules exist, so adding a module to `Modules` changes nothing it asserts.
`TestModules_SeedOnlyAndHubWideFlags` in the same file spells out the hub-wide set as a literal and fails for the new `HubWide: true` module.
`cmd/lym/configstrictness_test.go`'s `configStrictnessDegradingSet` pins every package that loads with `LoadOrTemplate`, and a new package that does so is missing from it.
The landed commit edited both of those and not `TestNames`, so the card's targets differ from the work its Intent describes.

### [MEDIUM] doc.go files under Extends that the cards rewrite, mostly with no stated change

**Location:** `_lym/plan/23-raddle-recipe.md`, `_lym/plan/25-pawl-raddle-handoff.md`

**Issue:** The rubric reserves `Extends` for adding to a ref without changing anything already there.
Card 23 extends six `doc.go` files; five of them state the recipe set in a sentence that adding `raddle` rewrites: `internal/shedrun/doc.go` line 5 ("batten, darn, motif, pawl, sampler and tapestry"), `internal/loomrecipe/doc.go` (lines 34, 48 and 63 list the recipes), `internal/shedcli/doc.go` line 12, `internal/runprofile/doc.go` (lines 3 and 12, including the defaults sentence), and `internal/loomcli/doc.go` line 25, whose "a recipe with no Finalize row, a motif run, makes its entry a Sketch" turns false for a raddle run.
The card's Intent describes only the `loomcli/doc.go` change, so an implementer is told nothing for the other four.
Card 25 extends `internal/pawlengine/doc.go`, whose line 23 lists the files under `.lym/beam/` that the new hand-off marker joins, and whose Beam-Up line 8 lists the steps; the Intent says nothing about the file.
`internal/shedrecipe/doc.go` has no recipe list and takes per-row sections, so it stays under Extends.

## Rejected

- `create-already-exists` blocking findings on cards 8 to 12 in the baseline `validate-plan --overlay` run: they come from the landed tree, the gate already owns them, and Webster's auto-rebaseline clears them.
- Missing `DependsOn` or `Produces`, and a missing `ImpactSummary` on a `Prosa` card: both are in the rubric's Do-not-flag list.
- `PATTERN-tolerant-records` for `store.json` and `approval.json`: the pattern bans `DisallowUnknownFields` and requires rewrites through `internal/state`; no card prescribes either violation, and `store.json` is written once.
- The motif-keyed branch in `internal/battencli/cleanup.go` (`clearMotifRun`) and the recipe list in `internal/boardcli/cli.go`'s upsert help: neither is part of what D1 asks batten to do for a raddle run, whose slug differs from every filed slug (D5).
- The six round-2 carry-over items listed in the decision record: each is carried by its card as the record's text requires (loomshed allowlist in cards 19 and 22, `Optional` in card 21, the hand-off marker target in card 25, the recipe-key constant in card 23, the tolerance mechanism in card 12, the halt reachability in card 22).
