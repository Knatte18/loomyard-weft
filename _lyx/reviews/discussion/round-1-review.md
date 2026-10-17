---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D5
    summary: D5 never says a review row naming a fan setting whose value is null or empty still records its round's fans, while the fix row naming fixer_fan_setting requires a recorded round, so the absent-means-no-entry reading fails every default-config fix round
    fix: In D5, after the sentence "A null, an empty string and an all-empty list count as absent, and a null auto-accept is false.", add "A BurlerReview row that names reviewer_fan_setting or fixer_fan_setting records its round's fans whatever the value, as a present wiring entry does today; absent means only that the round runs solo or with the single fixer and that the value does not conflict with a recipe profile fan. A BurlerFix row naming fixer_fan_setting requires that recorded round." In Acceptance criteria, change the scenario "a null, empty or all-empty fan setting runs solo, ..." to begin "a null, empty or all-empty fan setting runs solo and still records the round, so its segment's fix row runs the single fixer from the record, ...".
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D8
    summary: D8 says the darn wiring builds the Darn-Review entry from darn.yaml, but its Bouncer models come from loom.yaml's webster judge pair and its model fallback comes from the shared lists in Env.ReviewModels, which Scope removes
    fix: >-
      In D8, replace "and the darn wiring adds the `Darn-Review` entry from `darn.yaml`." with "and the darn wiring adds the `Darn-Review` entry: its budget, hard cap and models from `darn.yaml`, each empty darn model key falling back to `loom.yaml`'s shared `reviewer_model` and `fixer_model` lists, which the wiring hands over directly now that `Env.ReviewModels` is gone, and its Bouncer seed and judge models from `loom.yaml`'s webster judge pair, as today."
  - id: F3
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D8
    summary: D8 gives the build path budgets from Segments and the status path budgets from its own read, but never requires both to use one perch table and one resolution, and never says what Routing does with a review segment missing from its budget map
    fix: In D8, after "The status and pause path and the `SegmentBounces` routing closure pass budgets from their own config reads, so the status path keeps its degrade-to-template read and gains no model resolution.", add "Those budgets come from the same segment-to-perch table and the same per-perch resolution (`loomengine.Config.Caps`, `darn.yaml`'s `review_max_bounces`) that fill `Segments`, so the routing a status read projects equals the one `New` and `NewDarn` apply; `Routing` and `DarnRouting` refuse a review segment their budget map lacks, naming the segment." Add an Acceptance criteria scenario "the status path's routing and the built shed carry the same MaxBounces on every review row for one loom.yaml".
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 and D6
    summary: D6 refuses a fan that is neither a string nor a list of strings, which a null is, while D5 counts a null fan as absent; and on a hub run the loader the wiring still runs refuses a null or empty-list fan before Setting reads it
    fix: In D6, change "a value of the wrong shape (a fan that is neither a string nor a list of strings, an auto-accept that is neither a bool nor null)" to "a value of the wrong shape (a fan that is neither null, a string nor a list of strings, an auto-accept that is neither a bool nor null)". In D5, after "A null, an empty string and an all-empty list count as absent", add "in what `Setting` returns; on a hub run the module loaders the wiring still runs refuse a null or empty-list fan first, so the rule reaches a standalone or test caller's lookup".
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria and Notes for the plan writer
    summary: The third acceptance criterion promises a new plain value needs no envkit change, but the Notes only say envkit.FullEnv must fill Setting, so a fake answering a fixed key set would force an envkit edit for every new recipe _setting key
    fix: In Notes for the plan writer, replace "`envkit.FullEnv` and `NilSeams` must fill `Setting` and `Segments`; `shedrecipe`'s own fixture keeps its local builder." with "`envkit.FullEnv` fills `Segments` for every review segment of the embedded recipes, and fills `Setting` with a lookup that answers any registered module's template key from that template, so a new recipe `_setting` key needs no kit change; `shedrecipe`'s own fixture keeps its local builder."
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md PATTERN changes
    summary: The new PATTERN entry says a resolved per-row value is a SegmentSettings field, but only the review entries read SegmentSettings and D3 ties it to review segments, so a resolved per-row value of a row outside a review segment, such as the stamped verify attempts of Darn-Write, has no permitted home
    fix: In PATTERN changes, and identically in the matching Declared PATTERN text, change "a resolved one is a `SegmentSettings` field" to "a resolved one of a review segment's row is a `SegmentSettings` field".
  - id: F7
    severity: NIT
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 Rationale
    summary: The rationale says a per-build snapshot keeps a segment on one value as one wire-time read does today, but the fans now come from the build-time snapshot while models, budgets and fan-name validation come from the earlier wire-time loader read
    fix: In D4's Rationale, replace "A per-build snapshot keeps the rows of one segment on one value, as one wire-time read does today." with "A per-build snapshot keeps the rows of one segment on one fan value; it is a later read than the wire-time loader read that validates fan names, and burler refuses a fan that no longer resolves when the round starts."
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D8
    summary: D8 has loomrecipe refuse a negative budget naming its config key, but loomrecipe is handed one resolved int per segment and cannot tell whether it came from the perch key or review_max_bounces
    fix: In D8, change "a negative budget stays refused, naming the segment and its config key" to "a negative budget stays refused, naming the segment and the perch's budget key and `review_max_bounces` as the keys to check".
  - id: F9
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Constraints
    summary: The last Constraints bullet restates the first acceptance criterion and D5's refusal-text change, and adds no requirement of its own
    fix: >-
      In Constraints, delete the bullet "A loom or darn run behaves as before: the same values reach the same rows; only loom's fix-fan refusal text changes, per D5."
---

### [MEDIUM] A null or empty fan setting leaves fan recording undefined, and the fix row depends on it

**Location:** `_lyx/discussion/decision-record.md` D5

**Issue:** `burlerReviewEntry` builds `RoundFans`, and so records the round's fans, whenever the row's wiring map entry is present, even when the list names no fan (`internal/shedrecipe/entries_burler.go:111-120`).
The loom wiring fills an entry for every review row (`internal/loomcli/wiring.go:895-910`), and every perch fan key's template default is `""` (`internal/loomengine/template.yaml:19-26`), so a default loom run records solo rounds today.
The fix row is told `RequiresFans` from its fix-fan key (`entries_burler.go:182-184`), and the producer refuses a round whose record holds no fan names (`internal/shedadapters/burlerfix.go:137-154`).
D5 counts a null, empty or all-empty value as absent and says a row naming no setting behaves as an absent map entry.
It never says whether a row naming a setting with an empty value records its fans.
If the plan reads absent as no entry, a default `loom.yaml` gives a review row that records nothing beside a fix row that refuses every round.
All four lenses reached the same gap on their own.

### [LOW] The Darn-Review entry is not all from darn.yaml

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** Darn-Bouncer's seed and judge models are filled from loom.yaml's webster judge pair for every recipe (`internal/loomcli/wiring.go:913-919`).
The darn models resolve through `loomengine.ResolveDarnReview(env.ReviewModels, cfg, registry)` (`internal/loomcli/darnwiring.go:172`), whose fallback is the `Env.ReviewModels` field that `## Scope` removes.
D8's "from `darn.yaml`" names neither source, yet the first acceptance criterion requires the same Bouncer models and the same models.

### [LOW] The status path's budgets and the build path's budgets have no shared source

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** Today `perchMaxBounces` feeds both the full wiring's budgets and the status path's `reviewBudget` (`internal/loomcli/wiring.go:868-873`, `internal/loomcli/arm.go:262-279`), so they agree by construction.
Under D8, `New` reads each budget from `Segments` while the status path and the `SegmentBounces` closure resolve their own, with no requirement that both use one perch table and one resolution.
`Routing` today takes a run-wide fallback; under D8 it takes resolved per-segment budgets only, and the record does not say what it does with a segment missing from the map.
A drift makes status progress and the CIRCLING reason report a budget the engine does not apply, and no acceptance scenario checks that the two agree.

### [LOW] The null-fan rule contradicts D6 and is unreachable on a hub run

**Location:** `_lyx/discussion/decision-record.md` D5 and D6

**Issue:** D6's shape refusal covers "a fan that is neither a string nor a list of strings"; a null is neither, yet D5 and the acceptance scenario treat it as absent, and D6 exempts null only for auto-accept.
On a hub run the loaders the wiring still runs decode a null fan to an empty `FanList`, and `burlerengine.ValidateFanKeys` refuses an empty list (`internal/burlerengine/fankeys.go:61-63`, called from `internal/loomengine/config.go:1245`), so `Setting` never sees one there.

### [LOW] Acceptance criterion 3 depends on an envkit design the record leaves open

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, Notes for the plan writer

**Issue:** The loom and darn recipe tests build the embedded recipes over `envkit.FullEnv` (`internal/testkit/envkit/envkit.go:207`).
If its `Setting` answers only a fixed key set, D6 refuses each new `_setting` key at build, and adding one forces an `envkit` edit, which the third criterion rules out.
The Notes require `FullEnv` to fill `Setting` but not how it answers, so the criterion holds only under one choice.

### [LOW] The new PATTERN entry leaves resolved per-row values outside review segments with no home

**Location:** `_lyx/discussion/decision-record.md` PATTERN changes

**Issue:** The declared text says a resolved per-row or per-segment value "is a `SegmentSettings` field; never its own `Env` field".
`SegmentSettings` is keyed by review segment, D3 ties it to the Bouncer, BurlerReview and BurlerFix entries, and no other entry reads it.
A resolved value stamped on chosen rows outside a review segment, the class `DarnVerifyAttempts` belongs to (it is stamped on Darn-Write, in the PR-Review segment), would be barred from `Env` with nowhere else to go.

### [NIT] The snapshot rationale claims parity with a single wire-time read

**Location:** `_lyx/discussion/decision-record.md` D4 Rationale

**Issue:** `buildLoomShed` runs `wireLanding` before `newShed` (`internal/loomcli/sharedbootstrap.go:361-369`), so the build-time `Setting` snapshot is a later read than the wire-time `loomengine.LoadConfig` call, which validates fan names through `ValidateFanKeys`.
Today one read feeds fans, models and budgets alike, so the rationale's "as one wire-time read does today" is no longer accurate.
Burler's run-time refusal of an unresolved fan limits the effect.

### [NIT] A negative-budget refusal cannot name the key it came from

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** Today's refusals name the row or `review_max_bounces` (`internal/loomrecipe/loomrecipe.go:123-127`, `:138-143`).
Once `loomrecipe` is handed only a resolved int per segment, it cannot tell the perch key from the run-wide one.

### [NIT] A Constraints bullet restates the acceptance criteria

**Location:** `_lyx/discussion/decision-record.md` Constraints

**Issue:** The first acceptance criterion already holds "same values reach the same rows", and D5 already states the loom fix-fan refusal text change.
The bullet carries no requirement the plan writer would lose without it.

## Rejected

- interactions D3×D8, a segment with no `Segments` entry refused twice (by `New` and by the entry): not a defect; the duplicate refusal is harmless, and D3's still covers direct entry calls.
- `internal/battencli/wire_seams_test.go`'s `intentionallyNil` must gain `Setting` and `Segments`, since batten fills neither: a call-site enumeration, which the rubric's `Do not flag` list forbids.
- Focus item on D4's guard bound against the `CategoryNone` local-layer scenario: not a defect. `configengine.Layers` runs `ValidateLocal` and refuses a whole local file that holds a guard key (`internal/configengine/layers.go:224-265`, `:320-326`), and `run_settings` reach the pair only through that layer, so the bound holds by refusal and a `CategoryNone` local key is answered.
- Focus item on D5 against `## Scope`: not a defect. Every replaced field has a successor: models, model keys, Bouncer models, budgets and hard caps go to `SegmentSettings` (D1), and the fans, auto-accept and the `FixFan` half of `RowModelKeys` go to `_setting` keys (D5).
- Focus item on D3's removed `ReviewModels` fallback: not a defect. It tightens the guard into a refusal, so a row with no entry skips nothing, and D8's wiring fills the four loom segments and Darn-Review, every review segment the embedded recipes hold.
- Focus item on the `PATTERN-config-strictness` background change: not a defect. `configreg` becomes a strict caller of `configengine.Load`, and the background's strict-caller list must say so.
