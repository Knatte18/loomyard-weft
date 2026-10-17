# shedrecipe: per-segment settings and one told setting lookup

`shedrecipe.Env` carried a hand-threaded field per config value, and each new value touched `loomcli` wiring, the `Env` field, the reading row and `envkit`.
This change folds the per-row review maps into one per-segment struct and adds one told lookup, so a new plain config value needs a recipe key and a template key and no Go wiring change.
A loom run and a darn run resolve the same models, model keys, Bouncer models, bounce budgets, hard caps, fans and auto-accept flags per row as before.

## What changes

- `Env.Segments map[string]SegmentSettings`, keyed by the recipe's `segment:` value, replaces `RowReviewModels`, `RowModelKeys`, `RowMaxBounces`, `ReviewMaxBounces`, `RowHardMaxBounces`, `RowBouncerModels`, `DarnReviewMaxBounces` and `ReviewModels`.
  A segment's rows share one entry, so a new review segment is data.
- `Env.Setting`, a `func(module, key string) (any, error)`, replaces `RowClusterFans`, `RowFixFans`, `RowAutoAccept` and the fix-fan half of `RowModelKeys`.
  It names no `shedrecipe` type, so `shedrecipe` still imports no config package.
- `configreg.SettingLookup` fills it.
  It resolves the module's spec, loads the stack with strict `configengine.Load` over the told `Dirs`, and answers from a per-lookup snapshot; the wiring makes one lookup per recipe build.
  An unregistered module, or a key absent from the module's embedded template, is an error.
- `shedrecipe.Constructor` takes a `RowIdentity` (row name and segment) instead of the bare name, and `shedbuild.Build` fills it.
- The loom and darn recipes name the settings their review rows read: `reviewer_fan_setting`, `fixer_fan_setting` and, on loom's Bouncer rows, `auto_accept_setting`, each a `module.key` string.
  Darn names no auto-accept setting, so Darn-Bouncer's auto-accept stays false.
- `loomrecipe.New`, `NewDarn`, `Routing` and `DarnRouting` take bounce budgets keyed by segment.
  The status and pause path builds the same budgets from the same segment-to-perch table, so its routing equals the one the built shed applies and it still degrades to the template when no config file exists.
- No `loom.yaml` or `darn.yaml` key is added, renamed or removed.

## Design decisions

- A Bouncer, BurlerReview or BurlerFix row with no segment, or a segment with no `Segments` entry, is refused at build.
  The run-wide `ReviewModels` fallback for such a row is gone.
- Every setting a row names is read at construction, the fix row's included.
  A reference with no dot, a nil `Env.Setting`, an unknown module or key, a value of the wrong shape and a lookup error are all build refusals that name the row, the key and the reference.
- A null, empty or all-empty fan counts as absent, and a null auto-accept is false.
  A review row naming a fan setting records its round whatever the value, so the segment's fix row runs the single fixer from the record.
  A config fan beside a recipe profile fan is still a construction error.
- The fix row's unrecorded-round refusal cites `<key> in <module>.yaml` for loom and darn alike; loom's refusal gains the file name.
- A test over the embedded recipes requires a segment's rows to name the same fan setting, and the BurlerFix row to name `fixer_fan_setting` exactly when its BurlerReview row names it or sets a profile `fix-fan`.
  Without it, omitting the fix row's key would bypass that refusal.
- Out of scope: run-wide `Env` scalars, batten wiring (no batten row reads a setting yet), row entry gates and the `Command` producer.
- `PATTERN.md` gains `PATTERN-recipe-setting-lookup`, and the strict-caller list in `pattern/PATTERN-config-strictness.md` gains `configreg`.
  `shedrecipe/doc.go`, `loomrecipe/doc.go`, `contracts/specs/shed-recipe-spec.md` and `docs/research/haiku-judge.md` follow the new field names.

## To check by hand

- `PATTERN.md`: to fit the overview headroom, ten entries unrelated to this change were shortened.
  Two keep a restored qualifier ("orch stencils are exempt" in `PATTERN-edit-directive`, "never duplicated" in `PATTERN-testkit`).
  Confirm that the other eight lose nothing their `pattern/` background files do not carry.
- `internal/burlerengine/fix.go:322` still says "remove <key> from the profile" when the fixer's model has no config key, though the key is now a bare `loom.yaml` setting.
  `internal/shedadapters/burlerfix.go:142` likewise no longer says `in loom.yaml`.
  Both were left unchanged because rewording `burlerengine` refusals was out of scope.
- A key rename whose migration missed the hub layer (#709) makes `Setting` answer the new key's template default with no warning, as the module loaders do today.
- A recipe author can name a valid key of the wrong meaning; only the agreement test and review catch it.
- `row-entry-gate` owes the batten `Setting` fill and replaces the darn edge rewrite whose budget source this change rewires.
