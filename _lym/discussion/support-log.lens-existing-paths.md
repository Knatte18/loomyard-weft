```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already read or write the state each decision adds or changes, and checked whether each decision's result survives them.

**Findings**

- **MEDIUM** — D3 and D11: the block row has no engine and no registry entry, but several code paths look up every row's engine.
  - Evidence: `shedbuild.Build` calls `shedrecipe.Lookup(row.Engine)` for every row. `loomrecipe.RecipeEngines` collects `row.Engine` into the set the cross-consumer coverage guard trusts. `Rows`, `HasEngine`, `EngineRow` and `GateFixerSetting` iterate `parsed.Producers`.
  - Under D11 every accessor reads the expanded recipe, so the `Cartoon` row reaches all of these. It would add an empty engine name to `RecipeEngines`, and `Build` would fail its lookup unless it has a special case.
  - D3 says the block row "builds into a ProducerDef whose producer returns Done at once" but not who constructs that producer.
  - Constructing a producer outside the registry departs from `PATTERN-shed-recipe-registry` ("one `Constructor` map"). The record's PATTERN change adds only `BlockGates`, not the block row's producer.
  - Fix: either register a `Block` entry in the `Constructor` map and say the expander stamps it as the block row's engine, or state the exception and add it to the PATTERN change. Also state that engine-keyed accessors skip or recognise the block row.
- **MEDIUM** — D6 and D11: goto entries and `lym loom status` need the block path in the routing projection, and the record never says the projection carries it.
  - Evidence: `Goto` and `Relaunch` append `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto}` (goto.go:136, goto.go:250). They take their producers from a `Routing.Producers` projection, and `GotoRequest` reads only `Name`, `Segment`, `OnDone` and `OnStuck`.
  - `lym loom status` builds its view from `loomrecipe.Routing`, not from a built `Shed`.
  - D6 promises the block path on goto entries and in status lines. D11 adds "a routing projection from an already-expanded recipe" but does not say it copies the new `Block` field, or that `GotoRequest` reads it.
  - As written, goto entries would carry no block and status would have none to show.
  - Fix: state in D6 or D11 that the routing projection carries each row's block path and that goto and relaunch read it from there.
- **MEDIUM** — D3: the block row's exit must live somewhere after expansion, and the loom contract check walks `on_done` on the parsed rows.
  - Evidence: `checkLoomContract` and `onDonePath` follow `Row.OnDone` before `Build`. The record and build rows must lie on that path, so the expanded `Cartoon` row's `on_done` has to name `Discussion-Write`.
  - D2 implies this ("block row's Done goes to the sub-recipe's entry"), but the block's original `on_done` then needs a non-YAML field on `shedbuild.Row` for `Build` to copy into the skip target. The record names the skip target only on `ProducerDef`.
  - If the plan keeps the exit in `on_done` and resolves the entry in `Build`, the contract check refuses `arras`.
  - Fix: say expansion moves the block's `on_done` into a skip field on the row and rewrites `on_done` to the sub-recipe's entry, before any contract check or routing projection.
- **LOW** — D8: the record states the right path, so this is a confirmation.
  - Evidence: `battencli.prepareRunSettings` calls `profiles.Default(settings.Recipe)` and `profiles.Settings(name, settings.Recipe)` with the raw recipe name (internal/battencli/runsettings.go).
  - `runprofile.decodeDefaults` and `configreg.checkProfiles` key by recipe name too, but only check the file's own blocks, so they still read correctly.
  - The fix only works if every reader goes through `profile_recipe`. Seed-Child is the one that matters, and D8 already says "every reader … through one accessor". It survives once that accessor replaces the raw name there.
- **LOW** — D10: `lym loom peers` positions shift by one row between recipes.
  - Evidence: peers.go `recipeRow` gives each peer's 1-based row in its own recipe. An `arras` peer at Drift-Baseline reads one row later than a tapestry peer at the same step, because of the `Cartoon` row.
  - darn and tapestry peers already differ this way, so this is not new behaviour. It is worth one line if peers are compared during the bench.
- **LOW** — D10: the report phase table needs a phase for the `Cartoon` row.
  - Evidence: `reportengine.recipePhaseTables` maps rows to phases only for tapestry and darn. An `arras` entry copied from tapestry's leaves `Cartoon` with no phase.
  - The record's Notes name the file but not this mapping. It is plan-level detail.

**Paths traced, per decision**

- **D2 and D3:**
  - `enter` skip routing (entry.go `persistRouted(def.OnDone)`, `findProducer`): breaks without the skip target, which D3 covers. Survives with it.
  - `EnteredRow` and goto bypass: survives.
  - Loom contract `onDonePath`: survives only if expansion rewrites `on_done` (finding 3).
  - `shedengine` validate, which requires `OnStuck` in the same segment: survives, since inlined edges stay inside their block.
  - Progress `mainLine` following `OnDone`: survives.
- **D4:** the stuck halt at a row with no `on_stuck`, resume, and episode counting (`episodeStuckCount`, `segmentEnders`): survives, since names and segments are unchanged.
- **D5:** with `always` as a nil gate, `enter` returns early and never marks the row entered. That is harmless and survives.
- **D6:**
  - run.go:352, entry.go:51 and the reroute.go appends: survive once each site stamps `def`'s block.
  - Goto and relaunch appends: break unless the projection carries the block (finding 2).
  - `RenameProducers` touches only producer names: survives.
- **D7:** `loomSegmentPerches`, `segmentBudgets` and `stampZeroRounds` are keyed by segment name: survive.
- **D8:** Seed-Child's `prepareRunSettings`: survives only through the accessor (finding 4). `configreg.checkProfiles` and boardcli's profile check: survive.
- **D9 and D10:**
  - Discussion gate, `RecordEngine`, `CommitDiscussion` and arbiter keys are keyed by row or engine: survive.
  - `AmendmentTarget` reads the parent's key: survives.
  - `RetiredRows`: none for `arras`, survives.
  - `boardRecipe` and the supported child list in wire.go: a name site, covered by D10.
  - Peers rows: finding 5. Report phase table: finding 6.
- **D11:** the engine-keyed accessors and `RecipeEngines` break on the engineless block row (finding 1). `Routing` must carry the block path (finding 2).

**Extra tool calls**

- run.go lines 630–660, `validate`, and the runprofile lookups: to trace episode enders and find who reads profile blocks by recipe name.
- runprofile.go `Settings`, `Default` and `decodeDefaults`, plus their callers: for the exact lookup paths D8 changes.
- battencli/runsettings.go and configreg `checkProfiles`: to confirm Seed-Child uses the raw recipe name.
- peers.go row use, and the `Relaunch` and `RenameProducers` heads: for the peer row positions and status rewrite paths.
- goto.go, reroute.go and run.go history appends, plus reportengine/timingspans.go: for every site that appends history and for the report phase table.
