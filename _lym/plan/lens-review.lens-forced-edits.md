```yaml
lens: forced-edits
outcome: reported
```

Lens forced-edits: what each card's changes force outside the members it names.

Findings:

1. BLOCKING. Card 9 (D4, D7): the two new loom keys are not classified in loom's `ConfigSpec`.
   - Card 9 adds `raddle` and `raddle_auto_approve` to `template.yaml`, but it does not list `internal/loomengine#ConfigSpec`, whose `Categories` map classifies every key (`internal/loomengine/config.go:567-579`, e.g. `"discussion": none`, `"discussion_gate_fixer": none`).
   - `internal/configreg#TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go:75`) fails on an unclassified template key. Card 7 lists it only under Uses.
   - The category is itself a choice the plan must make. The `ConfigSpec` doc (`config.go:561`) bars from the local layer the keys "that can switch off or hollow a review or approval gate". `raddle_auto_approve` switches the operator approval off, so `guard` fits its rule.
   - But card 23 writes `loom: {raddle_auto_approve: false}` into every profile's raddle block, and D7 says "a run profile's raddle block may set it". Profiles are layered into the pair's local config, so with `guard` the profile block is refused.
   - Fix: add `ConfigSpec` to card 9's Edit list, and state each key's category and how the profile block survives it.

2. MEDIUM. Card 5 (D12, "every agent session a run spawns"): batten's runners get no usage hook.
   - Card 5 installs the hook only in `loomCLI.wire` and `gatecli#newStrandRig`.
   - Batten builds its own runners for its driver and judge agent sessions: `internal/battencli/wire.go:1287` and `internal/battencli/drive.go:462` (`shuttleengine.NewRunner`). Sessions started there write no usage file.
   - `internal/loomcli/looparm.go:122` builds a runner only for `StopStepRuns`. Stopped sessions are reported through `stopRecorded` (card 3), so that runner needs the hook too, or a watchdog-stopped session goes unrecorded.
   - No card lists these three sites.

3. MEDIUM. Card 8 (D13, "in dollars and in active hours"): hours depend on timing tables that select only tapestry and darn.
   - Card 8 joins hours from `ComputeTiming`, whose selection takes only `runRecipes = {tapestry, darn}` (`internal/reportengine/runs.go:24`). Its phase tables likewise hold only tapestry and darn (`internal/reportengine/timingspans.go:70-73`, `recipePhaseTables`).
   - So every motif, raddle, pawl and sampler cell gets no hours at all, while D13 reports hours per recipe.
   - No card extends `runRecipes` or `recipePhaseTables`, or gives raddle's rows a phase (the motif rows map to `PhaseDiscussion` at `timingspans.go:81`).

4. LOW. Card 23 (D1): `internal/loomrecipe/names.go` doc comments go stale.
   - Comments at `names.go:63` (`RowNames`), `names.go:77` (`Rows`) and `names.go:101` enumerate the recipe names. So does the "All five recipes" comment on `RecipeEngines`.
   - The card edits `RecipeEngines` and `embeddedRecipes` but does not list `internal/loomrecipe#RowNames` or `internal/loomrecipe#Rows`, so their comments stay wrong.

5. LOW. Card 23 (D1): a test of the interrupt-policy coverage keeps a stale message.
   - `internal/loomrecipe/interruptpolicy_meta_test.go:25` holds the loop the card extends.
   - `interruptpolicy_meta_test.go:40` carries the message "none of tapestry, darn, sampler and motif has", which the card does not mention rewording.

6. LOW. Card 23 (D1): the engine-to-producer shape table may need the raddle rows.
   - `internal/loomrecipe/shape_test.go:85` pins row name → engine → producer type (the Motif row is there).
   - Card 23 does not list it. If that table must cover every assembled row, Raddle-Write and Raddle-File need rows.

7. LOW. Card 23 (D1) and card 21 (D4): the new role name may need a constant.
   - `PATTERN-agent-name` says "a spawned role is its module's constant".
   - Card 21 gives the seat table the role prefix `raddle` but declares no role constant, and does not check it against `internal/agentname`.

Tool calls:
- Grep for callers of `checkNewSlugs`, `newSlugError`, `applyDraftRule`, `Promote` and `validateLabels`, the motif and recipe lists, the usage "OutcomeDone" comments and the `testtiming` reasons. My context lacked the callers outside the cards.
- Read `TreeRunRels`, loom's `ConfigSpec` categories, the `looparm` runner, batten's runner sites and the `cmd/lym` test list. I needed the record layout, the key classification and the runner construction sites.
- Grep for CLI pins, the `shape_test` and interrupt-policy meta tests, the `configreg` tests and the timing phase tables. I needed the test tables the new recipe and keys feed.
