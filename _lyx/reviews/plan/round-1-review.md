---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/04-review-rows-read-segments.md, Intent, Settings (D5, D6) paragraph
    summary: Card 4 adds the reviewer_fan_setting, fixer_fan_setting and auto_accept_setting recipe keys, a new routing edge from a row to any module.key, but its Intent never says what a row can now read or what bounds it
    fix: In card 4's Intent, after the Settings (D5, D6) paragraph, add one Bound sentence. It says a _setting key lets a row read any template key of any registered config module, a guard-category key included, through Env.Setting. It names the bounds - only rows of the recipes embedded in the binary name such keys; the lookup is read-only and answers only template keys of registered modules (card 1); a guard-category key never takes a local or run_settings value because configengine.Load refuses it; and TestShippedRecipes_FanSettingsAgreeWithinSegment pins each shipped row's exact reference, so a recipe edit that points a row at another key fails that test.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/03-wire-segments-and-setting.md, Create list
    summary: Card 3 puts two public symbols that other packages reuse, shedrecipe.SegmentSettings (read by loomcli and loomrecipe) and envkit.TemplateSetting (read by loomrecipe tests on card 4), on one card with the loomcli wiring; the granularity rule gives each its own card
    fix: Split card 3 into three cards with lyx loom insert-card, keeping their order before card 4. (a) shedrecipe-segment-settings creates plan:internal/shedrecipe#SegmentSettings, edits internal/shedrecipe#Env to add Segments and Setting, and edits internal/testkit/envkit#nilLegal to add both, so TestFullEnv_FillsEverySeam stays green. (b) envkit-template-setting creates plan:internal/testkit/envkit#TemplateSetting and edits internal/testkit/envkit#FullEnv and internal/testkit/envkit#TestFullEnv_FillsEverySeam. (c) The remaining card holds the loomcli wiring - loomSegmentPerches, darnReviewSegment, loomSegmentSettings, loomCLI.wire, loomCLI.wireDarn, wireDarnReview, TestWire_ReviewKeysReachTheEnv and TestWire_DarnRecipe - with its Uses list unchanged.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/04-review-rows-read-segments.md, Intent, Settings (D5, D6) paragraph
    summary: The nil Env.Setting refusal inherits the shared way forward that names the recipe key to fix, but editing the recipe cannot make progress here; dropping a fix row's fixer_fan_setting also breaks card 4's own D7 presence test
    fix: In card 4's Settings paragraph, change the sentence on way forwards to give the nil-Setting refusal its own way forward. It names filling Env.Setting in the caller that builds the recipe, for example a trailing way-forward clause reading fill Env.Setting in the caller that builds this recipe. In TestBouncerEntry_ConstructionFailures, the nil-Setting row asserts that text.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/05-budgets-by-segment.md, Create list (segmentBudgets) and the checkSegmentBudgets bullet
    summary: The segmentBudgets head takes no way-forward argument, yet the Intent gives it different missing-segment text for loom and darn; checkSegmentBudgets' negative-entry text names a <perch>_max_bounces key that loomrecipe cannot derive, because the segment-to-perch table lives in loomcli
    fix: In card 5, change the segmentBudgets declaration head to `func segmentBudgets(segments map[string]string, env shedrecipe.Env, wayForward string) (map[string]int, error)` and state that New passes the segment-to-perch-table text and NewDarn the darn wiring's Darn-Review entry text. In the checkSegmentBudgets bullet, replace the derived per-perch key with generic text, as today's applyReviewBudget wording has it - for loom, "set the perch's <perch>_max_bounces, or review_max_bounces, in loom.yaml", and for Darn-Review, darn.yaml's review_max_bounces. Checked with validate-plan --overlay beside F6's fix - no new finding.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/05-budgets-by-segment.md, Intent, segmentBudgets and checkSegmentBudgets bullets, Darn-Review negative refusal
    summary: The darn negative-budget refusal replaces checkDarnReviewBudget's runnable way forward (run lyx config darn --set review_max_bounces=<n> from the prime, then lyx loom resume) with set review_max_bounces in darn.yaml, which drops the hub-wide command and the resume step
    fix: In card 5, state that the Darn-Review negative-budget refusals of segmentBudgets and checkSegmentBudgets keep today's way forward from internal/loomrecipe/darnrecipe.go checkDarnReviewBudget - run "lyx config darn --set review_max_bounces=<n>" from the prime, then "lyx loom resume" - after naming the segment, the key and darn.yaml. In TestDarnRouting and TestNewDarn_Routing, the negative rows assert that clause.
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/00-overview.md, verify section
    summary: The verify lines run neither internal/battencli nor internal/battenrecipe, though both packages' tests read the envkit fields that cards 3 and 6 change (NilSeams over nilLegal, FullEnv), and card 3 rests its claim that battencli's wiring test needs no change on a test the plan never runs
    fix: In 00-overview.md's verify section, append ./internal/battencli ./internal/battenrecipe to the end of both lyx gate test lines, after ./internal/pattern. Checked with validate-plan --overlay, alone and beside F4's fix - no new finding.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/01-configreg-setting-lookup.md, TestSettingLookup rows
    summary: The card says every error row asserts the module, the key and way forward, but the guard-key and invalid-YAML rows reach configengine.Load's propagated error, which card 1 wraps only with the module name; ValidateLocal's text carries no way forward
    fix: In card 1's TestSettingLookup paragraph, split the assertion rule. The unregistered-module and unknown-key rows assert the module, the key and way forward. The guard-key and invalid-YAML rows assert the module name and the propagated cause - the guard key's name for the first, and an error for the second.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/04-review-rows-read-segments.md, Tests, TestBouncerEntry_ConstructionFailures, recipe value 3 row
    summary: That row is said to assert the error's row, key, reference and way forward, but configString's error (config key %q must be a string, got %T) carries no row in a direct entry call, no reference and no way forward
    fix: In card 4's TestBouncerEntry_ConstructionFailures bullet, say the recipe-value-3 row asserts only the key name reviewer_fan_setting and must be a string, as configString words it, and exempt it from the row, key, reference and way-forward rule in the Tests lead-in.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/04-review-rows-read-segments.md, Entries, BurlerFixProducer.Call bullet
    summary: The branch that leaves the clause out for an empty FixFanKey cannot be reached; after card 4, burlerFixEntry, the only production setter of RequiresFans, sets it exactly when fixer_fan_setting is named, which is exactly when FixFanKey is non-empty, and no planned test row has an empty FixFanKey
    fix: In card 4's BurlerFixProducer.Call bullet, delete the clause "with an empty FixFanKey the clause is left out", so the refusal always names p.opts.FixFanKey.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/04-review-rows-read-segments.md, Intent (burlerFixEntry) and ImpactSummary
    summary: The ImpactSummary leaves out internal/shedadapters, whose BurlerFixProducer.Call refusal text the card changes; and the changed loom FixFanKey value also reaches burlerengine's twill engine refusal (fix.go:322-327), which the card never names
    fix: In card 4's ImpactSummary, add that shedadapters' unrecorded-round refusal gains the setting's citation. In the burlerFixEntry bullet, add one sentence saying the <key> in <module>.yaml form also reaches burlerengine's twill engine refusal through RunOpts.FixFanKey, which darn already feeds in that form, so its text needs no change.
---

### [MEDIUM] Card 4 leaves the new `_setting` routing edge unbounded

**Location:** `_lyx/plan/04-review-rows-read-segments.md`, Intent, Settings (D5, D6) paragraph.
**Issue:** The plan-writer stencil, line 187, and the rubric's attack-surface bullet both require a card that adds a routing edge to state in its Intent what the edge lets through and what bounds it.
After card 4, a recipe row can read any template key of any registered module, guard-category keys included.
Card 1 states the lookup's own bound on `SettingLookup`.
Card 4 is the card that adds the recipe keys reaching it, and it names "the lookup bound" only as content for `doc.go`.
D4's bound sentence is the answer key here: read-only, template keys of registered modules only, and reached only by the recipes embedded in the binary.
The D7 test's exact-reference pin is a further bound.
Card 4's Intent never states any of these.

### [MEDIUM] Card 3 bundles two public surfaces with the wiring

**Location:** `_lyx/plan/03-wire-segments-and-setting.md`, Create list.
**Issue:** The spec's Card granularity section says a symbol that is part of a module's public surface, or reused across packages, gets its own card.
`envkit.TemplateSetting` is an exported kit function, and card 4's loomrecipe escalation tests reuse it.
`shedrecipe.SegmentSettings` and the two `Env` fields are shedrecipe's public surface, read by loomcli and loomrecipe.
Card 3 puts both on one card with the loomcli perch table and the darn wiring.
Each of the three parts can be reviewed on its own.
The `nilLegal` edit has to travel with the `Env` fields, or `TestFullEnv_FillsEverySeam` fails at that card.

### [LOW] The nil-`Env.Setting` refusal points at the recipe

**Location:** `_lyx/plan/04-review-rows-read-segments.md`, Settings (D5, D6) paragraph.
**Issue:** The card gives every settings refusal the way forward "names the recipe key to fix".
For a row that names a setting while `Env.Setting` is nil, the caller's wiring is at fault, not the recipe.
Removing the key would also break D7's presence agreement for a fix row.
`refusal-spec.md` classes nil deps as a fail-loud wiring guard, so this is a misleading message, not a `PATTERN-refusal-way-forward` violation.

### [LOW] `segmentBudgets` and `checkSegmentBudgets` cannot word what card 5 prescribes

**Location:** `_lyx/plan/05-budgets-by-segment.md`, the `segmentBudgets` declaration head and the `checkSegmentBudgets` bullet.
**Issue:** The `segmentBudgets` head carries no way-forward argument, yet the Intent gives it different missing-segment text for the loom and darn recipes.
`checkSegmentBudgets` receives only a segment-keyed `map[string]int`.
`loomSegmentPerches` lives in `internal/loomcli`, so `loomrecipe` cannot map `Plan-Review` to `plan_max_bounces`.
Today's `applyReviewBudget` text, `internal/loomrecipe/loomrecipe.go:164`, already names the key generically.

### [LOW] The darn negative-budget refusal loses its runnable command

**Location:** `_lyx/plan/05-budgets-by-segment.md`, `segmentBudgets` and `checkSegmentBudgets` bullets.
**Issue:** `checkDarnReviewBudget`, `internal/loomrecipe/darnrecipe.go:90-95`, names `lyx config darn --set review_max_bounces=<n>` from the prime, then `lyx loom resume`.
Card 5's replacement text, "set review_max_bounces in darn.yaml", drops that runnable form, which the hub-wide darn file needs.

### [LOW] The verify omits the batten packages that read the changed kit

**Location:** `_lyx/plan/00-overview.md`, verify section.
**Issue:** `internal/battencli/wire_seams_test.go:108` runs `envkit.NilSeams` over batten's wired `Env`.
`internal/battenrecipe/fixture_test.go:18` builds from `envkit.FullEnv`.
Cards 3 and 6 change `nilLegal` and `FullEnv`, and card 3's Intent rests on battencli's test staying green.
`go build ./...` compiles no test file, so neither package's tests run at any gate the plan sets.

### [LOW] `TestSettingLookup`'s error assertion cannot hold on the propagated-error rows

**Location:** `_lyx/plan/01-configreg-setting-lookup.md`, TestSettingLookup rows.
**Issue:** `ValidateLocal` refuses a local guard key with `key %q is guard and cannot be set in a local override` (`internal/configengine/layers.go:256-258`), and a YAML decode error carries no way forward either.
Card 1's step 3 wraps those errors only with the module name, so the blanket "module, key and way forward" assertion fails on those two rows.

### [LOW] The `configString` row's assertion cannot be met

**Location:** `_lyx/plan/04-review-rows-read-segments.md`, Tests, TestBouncerEntry_ConstructionFailures.
**Issue:** `configString` returns `shedrecipe: config key %q must be a string, got %T` (`internal/shedrecipe/config.go:28-31`).
The row name comes only from `shedbuild.Build`'s wrap (`internal/shedbuild/build.go:44-45`), which a direct entry call skips.
The error holds no reference and no way forward.

### [LOW] The empty-`FixFanKey` branch has no input that reaches it

**Location:** `_lyx/plan/04-review-rows-read-segments.md`, Entries, BurlerFixProducer.Call bullet.
**Issue:** `RequiresFans` is set only at `internal/shedrecipe/entries_burler.go:184`, inside `burlerFixEntry`.
Card 4 sets `RequiresFans` and `FixFanKey` from the same named setting.
Every planned unrecorded-round row of `TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview` sets `FixFanKey`.
So the rubric's reachability bullet catches the prescribed branch: it has no named input and no named test that reaches it.

### [LOW] Card 4 does not list every reader of the changed fix-fan citation

**Location:** `_lyx/plan/04-review-rows-read-segments.md`, burlerFixEntry bullet and ImpactSummary.
**Issue:** The ImpactSummary names shedrecipe, shedbuild and loomrecipe, while the card also changes `internal/shedadapters`' refusal text.
`internal/burlerengine/fix.go:322-327` renders `opts.FixFanKey` into the twill engine refusal, which the card never names.
That refusal already receives `darn_fixer_fan in darn.yaml` from the darn wiring (`internal/loomcli/darnwiring.go:182`), so the change is wording only.
The card should still say so.

## Rejected

- guard-reach's production-unreachable guards: `rowSegment`'s two branches, and the negative-budget refusals of `segmentBudgets` and `checkSegmentBudgets`. D3 and D8 prescribe each one, a named test reaches each, and `refusal-spec.md` classes nil or missing wiring as a fail-loud wiring guard. The lens itself raised this only as a note.
- Card 4's Create list (`rowSegment`, `settingRef`, `settingReference`, `fanSetting`, `boolSetting`): private helpers bundled into the card of the surface they serve, which the Granularity bullet requires. Listing each as a Create glyph is what the stencil asks of a new symbol, so this is not a defect.
- A delete on the same card as the removal of its last caller (cards 4, 5, 6): the spec's granularity section asks for two cards, but stencil line 179 allows a delete "on the same or an earlier card", and the symmetry rule bars flagging what the writer's stencil allows.
- Test economy: `TestShippedRecipes_FanSettingsAgreeWithinSegment` pins a rule that `TestShippedRecipe_ClusterExcludesAgreesWithPartnerFan` does not cover (cluster_excludes against the profile fan). It also folds the shipped-recipe pin and the cases into one test, where the existing rule uses two. `fakeSetting` is a local fixture, because shedrecipe's internal tests cannot import envkit without an import cycle. No planned test function repeats a covered behavior.

Other focus checks, verdicts only:
- Card order: every `plan:` Use is created by an earlier card, and cards 5 and 6 use nothing a later card creates.
- Re-sign arrows: present on every re-signed member, `Constructor` and every entry on card 2, and `Routing`, `DarnRouting`, `applyReviewBudget`, `stampZeroRounds`, `applyDarnReviewBudget`, `reviewBudget` and `darnReviewBudget` on card 5. `New`, `NewDarn` and the card 4 entries keep their signatures.
- Edit file paths: the Go file paths on Edit cards (`doc.go`) sit beside member glyphs, so the member-glyph bullet does not fire.
- ImpactSummary: those of cards 2 and 6 state real blast radii.
- Board entry: no card edits this task's board entry.
- Fidelity: D1 through D8 and every Constraint are carried, the `doc.go`, `PATTERN.md` and strict-caller obligations included. No card adds a `loom.yaml` or `darn.yaml` key, changes batten wiring, or touches row-entry-gate, command-producer or config-migrate-hub-layer.
