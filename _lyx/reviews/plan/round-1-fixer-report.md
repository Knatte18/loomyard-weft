# Fixer report — plan, round 1

Every finding's premise held against the code at HEAD; all ten are fixed.
`lyx loom validate-plan` passes after the fixes and after the self-check edits.

## Changes

- **F1** (fixed as proposed): card 6 (was card 4), after the Settings paragraph, gains a `Bound:` sentence: a `_setting` key reads any template key of any registered module, guard keys included, through `Env.Setting`; only embedded recipes name such keys; the lookup is read-only over template keys of registered modules (card 1); a guard key never takes a local or `run_settings` value because `configengine.Load` refuses it; `TestShippedRecipes_FanSettingsAgreeWithinSegment` pins each shipped row's reference.
- **F2** (fixed as proposed, one deviation): card 3 split with `lyx loom insert-card` into card 3 `shedrecipe-segment-settings` (`SegmentSettings`, `Env.Segments`/`Env.Setting`, `nilLegal`), card 4 `envkit-template-setting` (`TemplateSetting`, `FullEnv`, `TestFullEnv_FillsEverySeam`) and card 5 `wire-segments-and-setting` (the loomcli wiring and its two tests).
  The later cards moved to 6, 7 and 8; every prose card-number mention in cards 2, 5, 6, 7, 8 and the overview (Card Index line 5, the "cards 5 and 8" shared decision) was updated by hand.
  Fixed differently on one point: the fix said card 5's Uses stays unchanged, but its `internal/configreg#Lookup` Use served `TemplateSetting`, which moved to card 4; card 5's Uses now name `plan:internal/configreg#SettingLookup` and `plan:internal/shedrecipe#SegmentSettings`, which its wiring calls and builds, and card 4 takes the `configreg#Lookup` Use.
- **F3** (fixed as proposed): card 6's Settings paragraph gives the nil-`Setting` refusal its own way forward, `fill Env.Setting in the caller that builds this recipe`, and the `TestBouncerEntry_ConstructionFailures` nil-`Setting` row asserts it.
- **F4** (fixed as proposed): card 7 (was card 5) re-signs `segmentBudgets` with a `wayForward string` parameter, `New` passing the segment-to-perch-table text and `NewDarn` the darn `Darn-Review` entry text; `checkSegmentBudgets`' negative refusal uses generic text: `set the perch's <perch>_max_bounces, or review_max_bounces, in loom.yaml` for loom.
- **F5** (fixed as proposed): card 7's Darn-Review negative refusals in both `segmentBudgets` and `checkSegmentBudgets` keep `checkDarnReviewBudget`'s `lyx config darn --set review_max_bounces=<n>` then `lyx loom resume` way forward; `TestDarnRouting`'s and `TestNewDarn_Routing`'s negative rows assert it.
- **F6** (fixed as proposed): both verify lines in `00-overview.md` end with `./internal/battencli ./internal/battenrecipe`.
- **F7** (fixed as proposed): card 1's `TestSettingLookup` assertion rule split: unregistered-module and unknown-key rows assert module, key and `way forward`; the guard-key row asserts module and guard key name; the invalid-YAML row asserts the module name.
- **F8** (fixed as proposed): card 6's recipe-value-3 row asserts only `reviewer_fan_setting` and `must be a string`, and the Tests lead-in exempts it.
- **F9** (fixed as proposed): card 6's `BurlerFixProducer.Call` bullet drops the empty-`FixFanKey` clause.
- **F10** (fixed as proposed): card 6's ImpactSummary names shedadapters' unrecorded-round refusal and its tests; the `burlerFixEntry` bullet says the `<key> in <module>.yaml` form also reaches `burlerengine`'s twill engine refusal through `RunOpts.FixFanKey`, which darn already feeds in that form.

## Self-check

- fix-consistency, LOW, `lens-review.md` keeps the old card numbers: disputed, `every fix worse`. The file records an earlier lens pass at the numbering it ran against; rewriting its card numbers would falsify that record, and no card or the validator reads it.
- fix-affected-set, LOW, card 6's Tests lead-in exempted only the `configString` row, though the segment-refusal rows carry no key or reference and `fakeSetting`'s lookup error no way forward: fixed; the lead-in now states what each row kind asserts (settings refusal, segment refusal, lookup error, `configString` row).
- fix-claims, LOW, card 4's "(card 6)" for budgets, though `MaxBounces` is first read on card 7: fixed; it cites the entries and `loomrecipe` and cards 6 and 7.
- fix-proposal, MEDIUM, F8's test exemption contradicted card 6's Settings paragraph ("Every refusal names the entry, the row, the config key and the reference"): fixed; the paragraph now states that a non-string recipe value keeps `configString`'s own text, naming only the key, and scopes the way-forward rule to the other refusals. Wrapping the error in `settingRef` instead was not taken, since the review's F8 asked for the narrower assertion and either resolves the contradiction.

## Disputed

None of the review's findings.

## Deferred

None.
