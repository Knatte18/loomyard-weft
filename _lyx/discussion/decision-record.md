# Decision record — shedrecipe-capabilities

## Goal

Stop hand-threading one `shedrecipe.Env` field per config value.
Fold the per-row review maps into one per-segment settings struct, so a new review segment is data and a new resolved segment value is one struct field.
Give `Env` one told setting lookup that a recipe row reaches by naming `module.key`, so a new plain config value needs a recipe key and a template key and no Go wiring change.

## Scope

In:

- `Env.Segments map[string]SegmentSettings`, keyed by segment name, replacing `RowReviewModels`, `RowModelKeys`, `RowMaxBounces`, `ReviewMaxBounces`, `RowHardMaxBounces`, `RowBouncerModels`, `DarnReviewMaxBounces` and `ReviewModels`.
- `Env.Setting`, a caller-filled lookup, replacing `RowClusterFans`, `RowFixFans`, `RowAutoAccept` and the `FixFan` half of `RowModelKeys`.
- The registry's `Constructor` learns the row's segment.
- The recipe changes in `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml` that name the settings the review rows read.
- The hub-side lookup in `internal/configreg`, and the loom and darn wiring in `internal/loomcli` that fills `Segments` and `Setting`.
- `internal/loomrecipe`'s budget stamping and routing, keyed by segment.
- The test kit (`internal/testkit/envkit`, the `shedrecipe` local fixture) and `internal/shedrecipe/doc.go`.

Out:

- Run-wide `Env` fields that are not per-row maps (`ReviewCirclingCheckpoint`, `ReviewTimeout`, `DarnVerifyAttempts`, `VerifyTiers`, `RenameTools` and the like).
- Batten's wiring: no batten row reads a setting in this task, so `internal/battencli` does not fill `Setting`.
- Row entry gates (`row-entry-gate`), the `Command` producer (`command-producer`) and the hub-layer migration bug (`config-migrate-hub-layer`).
- No `loom.yaml` or `darn.yaml` key is added, renamed or removed.

## Decisions

### D1 — Per-segment settings keyed by segment name

`Env.Segments map[string]SegmentSettings` replaces the per-row-name maps.
Its key is the recipe's `segment:` value (`Discussion-Review`, `Plan-Review`, `Webster-Review`, `Merge-Review`, `Darn-Review`).
`SegmentSettings` holds the values the caller must resolve before a row can use them:

- the reviewer and fixer round models (`burlerengine.RoundModels`), already fallen back to the shared `reviewer_model` and `fixer_model`;
- the config keys those models came from (review and fix `shuttleengine.ModelKey`), for engine refusals;
- the Bouncer's seed and judge models (`shedadapters.BouncerModels`);
- the resolved bounce budget, 0 meaning zero rounds, and the config key and file it resolved from;
- the resolved hard cap and the config key and file it resolved from.

The budget and hard cap resolve through `internal/reviewbudget`, the leaf loom and darn config already build that key family on, so the wiring adds no second resolution.

Rationale: every one of these needs a fallback chain, a registry resolution or a validation owned by the module's config code, which `shedrecipe` must not learn.
Keying by segment makes the three rows of a segment read one entry, so a new segment is one map entry.

### D2 — The constructor learns its row's segment

`Constructor` takes a small row identity carrying the row's name and its `segment:` value in place of the bare name; `shedbuild.Build` fills it from `shedbuild.Row`.
Every registry entry takes the new parameter; only the Bouncer, BurlerReview and BurlerFix entries read the segment.

Rationale: an entry cannot read `Segments` without knowing its segment, and the recipe already declares it on the row.

### D3 — A Bouncer, BurlerReview or BurlerFix row needs a segment and a `Segments` entry

Each of the three entries refuses at construction a row with an empty segment, or a segment with no `Segments` entry.
The refusal names the row and the segment, and its way forward is to put the row in a review segment or to wire that segment's settings.
The run-wide `ReviewModels` fallback for a row with no entry goes, because the caller resolves every segment's models.

### D4 — One told setting lookup

`Env.Setting` has the plain func type `func(module, key string) (any, error)`, naming no `shedrecipe` type, so a later caller can hand the same value to a package that may not import `shedrecipe`.
It returns the module's resolved value for `key` from the pair's config stack, within `PATTERN-config-layers`' local-layer bounds, so a guard-category key never takes a local or `run_settings` value.
`shedrecipe` never learns the config file format and imports nothing new.

The hub fills it from one new `configreg` function, which resolves the module's spec, loads the stack with strict `configengine.Load` over the told `configengine.Dirs`, and answers the key.
A hub-wide module such as `darn` resolves from the board, as `PATTERN-hub-wide-config` places it, because the wiring passes the same `Dirs` its module loaders use.
A module `configreg` does not register, and a key absent from the module's embedded template, are errors; "known" is checked against the template's keys, never the loaded document, which `Load` fills down to the template.

One lookup value loads each module at most once and answers every later key from that snapshot; the wiring makes one per recipe build.
A caller that wants a later read to see a changed file makes a new lookup.

Bound: the lookup is read-only, answers only template keys of registered modules, and is reached only by rows of the recipes embedded in the binary.
Fan names keep their `burler.yaml` validation from the module loaders the wiring still runs; the lookup checks shape only.

Rationale: `configreg` already maps module names to specs, and the function calls `configengine.Load`, keeping layer reading there.
A per-build snapshot keeps the rows of one segment on one fan value;
it is a later read than the wire-time loader read that validates fan names, and burler refuses a fan that no longer resolves when the round starts.

### D5 — A recipe row names the setting it reads as `module.key`

A row config value that references a setting is a string `module.key`, split at the first dot; the key's name on the row ends in `_setting`.
Step 2's lookup carries the plain single-key values the operator's step 1 listed among the per-row maps, the fan lists, so `SegmentSettings` holds only resolved values.
This task adds three:

- `reviewer_fan_setting` on the Bouncer and BurlerReview rows: the segment's per-round reviewer fan list, replacing `RowClusterFans`.
- `fixer_fan_setting` on the BurlerReview and BurlerFix rows: the segment's per-round fixer fan list on the review row, and on the fix row the key its unrecorded-round refusal cites, replacing `RowFixFans` and `RowModelKeys.FixFan`.
- `auto_accept_setting` on the Bouncer row: the perch's auto-accept flag, replacing `RowAutoAccept`.

The loom recipe names `loom.<perch>_reviewer_fan`, `loom.<perch>_fixer_fan` and `loom.<perch>_auto_accept`; the darn recipe names `darn.darn_reviewer_fan` and `darn.darn_fixer_fan` and no auto-accept setting, so Darn-Bouncer's auto-accept stays false.
A row naming no setting behaves as today's absent map entry: no fan from config, auto-accept false.
A BurlerFix row naming no `fixer_fan_setting` reads no recorded fans and runs the single fixer;
D7's presence agreement over the embedded recipes bounds that, so a fix row never drops a fixer fan its review row records.

A fan value is one fan name or a list of them, per round, read as `burlerengine.FanList` reads it, a scalar and a one-entry list alike.
A null, an empty string and an all-empty list count as absent in what `Setting` returns, and a null auto-accept is false;
on a hub run the module loaders the wiring still runs refuse a null or empty-list fan first, so the null rule reaches a standalone or test caller's lookup.
A BurlerReview row that names `reviewer_fan_setting` or `fixer_fan_setting` records its round's fans whatever the value, as a present wiring entry does today;
absent means only that the round runs solo or with the single fixer and that the value does not conflict with a recipe profile fan.
A BurlerFix row naming `fixer_fan_setting` requires that recorded round.
A config fan beside a recipe profile fan stays a construction error, as today.
The fix row's refusal cites the setting as `<key> in <module>.yaml` for loom and darn alike, so loom's refusal gains the file name it lacks today.

Rationale: the fans and the auto-accept flag are single keys with no fallback or resolution, the class the lookup is for, and `module.key` is the form `row-entry-gate` plans for its gates.

### D6 — An unknown or malformed setting is refused when the recipe builds

An entry reads every setting its row names at construction, the fix row's `fixer_fan_setting` included, so `loomrecipe.New` and `NewDarn` refuse before any row runs.
Refused, each naming the row, the config key and the reference:

- a reference with no dot or an empty module or key;
- a row naming a setting while `Env.Setting` is nil;
- an unknown module or key, from the lookup;
- a value of the wrong shape (a fan that is neither null, a string nor a list of strings, an auto-accept that is neither a bool nor null);
- a lookup error, such as an unreadable config file.

The way forward names the recipe key to fix or the template that lacks the key.
These reads run where the recipe builds, on verbs whose wiring already loads `loom.yaml` and `darn.yaml` strictly.

Rationale: a string lookup loses the compile-time check a struct field had, so construction is where the check moves.

### D7 — A segment's rows agree on a fan setting

A test over the embedded loom and darn recipes requires the rows of one segment that name `reviewer_fan_setting` to name the same reference, and likewise for `fixer_fan_setting`.
It also requires presence to agree within each review segment: the BurlerFix row names `fixer_fan_setting` exactly when the BurlerReview row names `fixer_fan_setting` or sets a profile `fix-fan`, the two routes by which the review row records a fixer fan, and the BurlerReview row names `reviewer_fan_setting` exactly when the Bouncer does.

Rationale: one wiring key fed both the Bouncer and the review row, and the wiring filled the fix row's fan key for every fix row, so they agreed in reference and presence by construction; the test keeps both agreements, so the fix row's unrecorded-round refusal cannot be bypassed by omitting its key.

### D8 — Wiring fills `Segments` from a perch table; routing takes budgets only

`internal/loomcli` builds each loom segment's `SegmentSettings` from one table mapping segment name to its perch, and the darn wiring adds the `Darn-Review` entry: its budget, hard cap and models from `darn.yaml`, each empty darn model key falling back to `loom.yaml`'s shared `reviewer_model` and `fixer_model` lists, which the wiring hands over directly now that `Env.ReviewModels` is gone, and its Bouncer seed and judge models from `loom.yaml`'s webster judge pair, as today.
`loomrecipe.New` and `NewDarn` read each review segment's budget from `Segments`; a review segment the recipe holds with no `Segments` entry is a wiring error naming the segment, and a negative budget stays refused, naming the segment and the key and file the budget resolved from: the key from `reviewbudget.Caps` and the file from the segment's module (`loom.yaml` or `darn.yaml`), as D1's `SegmentSettings` carries them.
`Routing` and `DarnRouting` take the per-segment budgets alone, keyed by segment, never a `SegmentSettings` map.
The status and pause path and the `SegmentBounces` routing closure pass budgets from their own config reads, so the status path keeps its degrade-to-template read and gains no model resolution.
Those budgets come from the same segment-to-perch table and the same per-perch resolution (`loomengine.Config.Caps`, `darn.yaml`'s `review_max_bounces`) that fill `Segments`, so the routing a status read projects equals the one `New` and `NewDarn` apply;
`Routing` and `DarnRouting` refuse a review segment their budget map lacks, naming the segment.
`Routing`'s way forward is to add that segment's row to the loom wiring's segment-to-perch table;
`DarnRouting`'s is to add the `Darn-Review` budget entry the darn wiring builds from `darn.yaml`'s `review_max_bounces`.

Rationale: a new perch becomes one table row and one recipe segment, and the lightweight verbs never refuse over models they do not use.

## Constraints

- `PATTERN-shed-recipe-registry`: the registry stays one `map[string]Constructor` behind `Lookup` and `Names`, with no `init()` and no `lyxcwd`; only the `Constructor` parameter list changes.
- `PATTERN-told-geometry`: `shedrecipe` stays told; `Setting` is a closure, so the package imports no config, `configreg` or CLI package.
- `PATTERN-config-layers`: only `configengine` reads and stacks layer files; the `configreg` function calls `configengine.Load`.
- `PATTERN-config-strictness`: the entry's text stands; its background file's strict-caller list gains `configreg`.
- `PATTERN-sole-parsers`: splitting a `module.key` value interprets a decoded `Config` value, as entries already do; `shedbuild` stays the recipe file's sole parser.
- `PATTERN-refusal-way-forward`: every new refusal in D3, D6 and D8 names its way forward, and a test reaches it.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing entry and conformance tests over adding new top-level ones where they fit.

## Auto-mode assumptions

- The operator's step 1 list is read as every per-row-name map on `Env`.
- Only the per-row maps move; migrating run-wide `Env` scalars is not asked.
- `module.key` is the reference form, matching `row-entry-gate`'s examples.

## Open risks

- `row-entry-gate` owes the batten `Setting` fill and replaces the darn edge rewrite D8 rewires; the plan should not add tests that pin that rewrite beyond its budget source.
- After a key rename whose migration missed the hub layer (`config-migrate-hub-layer`, #709), `Setting` answers the new key's template default with no warning, as the module loaders do today.
- A recipe author can name a valid key of the wrong meaning; only the agreement test in D7 and review catch it.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - fan-setting-presence-agreement-unpinned (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - unrelated-pattern-entries-rewritten (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- A loom run and a darn run resolve the same reviewer and fixer models, model keys, Bouncer models, bounce budgets, hard caps, reviewer fans, fixer fans and auto-accept flags per row as before, Darn-Bouncer's auto-accept false.
- The fields `## Scope` names as replaced are gone from `Env`.
- Adding a plain config value to a review row needs a template key, its category in the module's `ConfigSpec`, a recipe `_setting` key and the reading entry's own code, with no change to `Env`, `loomcli` wiring or `envkit`.
- Covered scenarios:
  - a recipe naming an unknown module, an unknown key, or a key only an on-disk layer holds, is refused at build, naming the row and reference, the fix row's reference included;
  - a malformed reference and a nil `Env.Setting` beside a named setting are refused at build;
  - a fan setting of the wrong shape and an auto-accept setting that is neither a bool nor null are refused at build;
  - a lookup error propagates as a build refusal;
  - a Bouncer, BurlerReview or BurlerFix row with no segment, or whose segment has no `Segments` entry, is refused at build;
  - a null, empty or all-empty fan setting runs solo and still records the round, so its segment's fix row runs the single fixer from the record, a null auto-accept is false, and a config fan beside a recipe profile fan is still refused;
  - a `CategoryNone` key set in the pair's local layer is the value `Setting` returns, and one lookup answers a second read of a module from its snapshot;
  - a fix row cites `<key> in <module>.yaml` when it refuses an unrecorded fan round;
  - the rows of each segment in the embedded recipes name the same fan settings, the BurlerFix row names `fixer_fan_setting` exactly when its BurlerReview row names it or sets a profile `fix-fan`, and the BurlerReview row names `reviewer_fan_setting` exactly when the Bouncer does;
  - a zero budget gives that perch zero rounds, a negative budget is refused naming the segment and the key and file it resolved from, and the darn perch's budget still rewrites its edge;
  - the status path's routing works with no `Segments` map and a missing config file, degrading to the template as today;
  - the status path's routing and the built shed carry the same MaxBounces on every review row for one loom.yaml;
  - `Routing` and `DarnRouting` given a budget map that lacks a review segment refuse, naming the segment, `Routing` pointing at the segment-to-perch table and `DarnRouting` at the darn wiring's `Darn-Review` budget entry.

## PATTERN changes

- `PATTERN-recipe-setting-lookup` — adds: Giving a shed recipe row a per-row or per-segment config value: a plain value is named `module.key` and read through `Env.Setting`, a resolved one of a review segment's row is a `SegmentSettings` field; never its own `Env` field.

## Notes for the plan writer

- The `Constructor` change touches every registry entry and every test calling an entry directly; `seam_enforcement_test.go` may pin the constructor shape.
- `envkit.FullEnv` fills `Segments` for every review segment of the embedded recipes, and fills `Setting` with a lookup that answers any registered module's template key from that template, so a new recipe `_setting` key needs no kit change; `shedrecipe`'s own fixture keeps its local builder.
- The test budget in `cmd/lyx/testdata/test-budget.yaml` may need a row change if new top-level tests land in `shedrecipe`.
- `PATTERN-recipe-setting-lookup` is held by review; no scan pins it.
