---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/01-production-fans.md:30
    summary: Card 1 says the template self-consistency test needs no change, but it pins the seeded lens and fan counts, which the card's new lenses and fans break
    fix: >-
      In card 1, add `- internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent` to the Edit list, beside `internal/burlerengine#TestResolveFan`.
      Replace the sentence that says the test needs no change with one saying the test raises its `wantLenses` and `wantFans` pins by the lenses and fans the card adds,
      adds the four new fans to its per-fan entry-count map, and keeps its defined-lens cross-reference check.
      Checked with `lyx loom validate-plan --overlay`: adding the target passes every check the baseline passes.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/plan/01-production-fans.md:11
    summary: Card 1 says the new lenses are written as emphasis, but the template test forces an emphasis sentence the bench lens texts lack and D4 closes
    fix: >-
      In card 1's Intent, replace "written as emphasis, never exclusion, per the template's header" with the choice made.
      Recommended: the new lenses keep the bench text, which ends in its own "Your list:" sentence and, per D4, reports only what the lens procedure produces,
      so the emphasis-suffix assertion in `TestConfigTemplate_DecodesAndIsSelfConsistent` skips the lenses the four production fans name while its "ignore " check still covers every lens.
      State that in the same Intent paragraph that lists the test under Edit (F1).
      The alternative, appending the emphasis line to all twelve new lenses, must also be stated, together with its departure from D5's "the bench's text" and from D4's "a fork reports only what its lens's procedure produces".
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/plan/08-loom-per-round-fan-keys.md:32
    summary: Card 8 makes the two new fixer fan keys local-layer guards but omits the configreg test that pins loom's local-denied key list
    fix: >-
      In card 8, add `- internal/configreg#TestModules_EveryTemplateKeyClassified` to the Edit list, beside `internal/configreg#TestLookup`.
      In the Tests paragraph, add that the test's `wantLocalDenied` entry for `loom` gains `discussion_fix_fan` and `plan_fix_fan`, in the sorted order the derived list has.
      Checked with `lyx loom validate-plan --overlay`, together with F1's fix: no blocking finding beyond the baseline.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/08-loom-per-round-fan-keys.md:63
    summary: Card 8 names the non-scalar list entry as a fanShapeError trigger in its spec row but plans a test row only for a mapping value
    fix: >-
      In card 8's `TestLoadConfig_Refuses` sentence, after the row whose `discussion_fix_fan` is a mapping, add a row whose `plan_fan` list holds a mapping entry,
      which reaches `fanShapeError` through the list branch of `FanList.UnmarshalYAML` and is refused naming the key and the way forward.
      `TestLoadConfig_Refuses` already carries both shapes for the model-spec keys ("mapping review value", "mapping inside a fix list").
      A card-text change only, so no plan check constrains it.
---

### [BLOCKING] Card 1 leaves `TestConfigTemplate_DecodesAndIsSelfConsistent` unedited though it pins the template's lens and fan counts

**Location:** `_lyx/plan/01-production-fans.md:30`, against `internal/burlerengine/config_test.go` (`TestConfigTemplate_DecodesAndIsSelfConsistent`).

**Issue:** The card closes with "`TestConfigTemplate_DecodesAndIsSelfConsistent` already checks every template fan names a defined lens and needs no change."
That test also holds `const wantLenses = 23` and `const wantFans = 6`, each compared with `len` of the decoded template, and a per-fan entry-count map for the six existing fans.
The card adds twelve lenses (`interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `affected-set`, `guard-reach`, `way-forward`, `fix-consistency`, `fix-claims`, `fix-way-forward`, `fix-proposal`, `fix-affected-set`) and four fans, so both count assertions fail.
`./internal/burlerengine` is in the plan's `verify:` in both tiers, so card 1's gate fails on a test the card tells its implementer to leave alone, and the test is not in the card's Edit scope.
No decision licenses leaving the pins stale, and the test is the one the template carries a `testtiming:keep` marker on for exactly these pins.

### [MEDIUM] The new lenses' required emphasis sentence is unresolved against D4 and D5

**Location:** `_lyx/plan/01-production-fans.md:11`, against `internal/burlerengine/config_test.go` (the emphasis loop of `TestConfigTemplate_DecodesAndIsSelfConsistent`) and `internal/burlerengine/template.yaml:1-31`.

**Issue:** The card writes the new lenses "as emphasis, never exclusion, per the template's header", taking their texts from the bench.
The same test requires every lens but `generic` to end with `Report anything else you notice too — emphasis, never exclusion.`; the bench texts end instead with their own `Your list: ...` sentence.
Appending the emphasis line changes the text D5 and D8 take from the bench, and it re-opens the off-lens reporting that D4 closes when it drops the off-lens BLOCKING escape so that a fork "reports only what its lens's procedure produces".
Skipping the assertion for the new lenses keeps D4 and D5, but then the test changes in a way the card never states.
The card chooses neither, so an implementer either breaks the test or edits a decision's text without a ruling.

### [BLOCKING] Card 8 omits the configreg test that pins loom's local-denied keys

**Location:** `_lyx/plan/08-loom-per-round-fan-keys.md:32`, against `internal/configreg/configreg_test.go` (`TestModules_EveryTemplateKeyClassified`, its `wantLocalDenied` map).

**Issue:** Card 8 makes `discussion_fix_fan` and `plan_fix_fan` guards in `ConfigSpec`, "so all four are refused in the local layer".
`configreg` derives each module's `LocalDenied` from the spec's guard categories (`configengine.LocalDenied`), and `TestModules_EveryTemplateKeyClassified` compares that derived list with a literal for `loom` that holds `discussion_fan` and `plan_fan` and neither new key.
After the card the derived list gains two keys and the test fails; the card edits `TestLookup` in the same package, which pins only the open maps, and not this test.
`./internal/configreg` is in `verify:` in both tiers, so card 8's gate fails.

### [LOW] The non-scalar list-entry trigger of `fanShapeError` has no test input

**Location:** `_lyx/plan/08-loom-per-round-fan-keys.md:63`

**Issue:** The card's spec row, `config: fan key not a fan list`, names two triggers: a key holding a mapping and a list with a non-scalar entry.
`TestLoadConfig_Refuses` gains one row for the mapping value only.
The two shapes leave `FanList.UnmarshalYAML` through different branches, as `ModelSpecList.UnmarshalYAML` does, and the model-spec keys test both ("mapping review value", "mapping inside a fix list").
The card's list-entry branch is a prescribed refusal path with no named input that reaches it.

## Rejected

- Focus 1, way forward and row of `fanShapeError`: the card states the way forward in `fanShapeError.Error()`, the key-naming wrap in `LoadConfig` carries it as it does for `modelSpecShapeError`, the spec row `config: fan key not a fan list` lands in the same card, and the mapping row reaches the error.
  Only the list-entry branch is open (F4).
- Focus 2, recipe fan with `cluster_excludes` true under the template defaults: card 7 counts an all-empty wiring list as absent in both `burlerReviewEntry` and `bouncerEntry`, so the review row runs the recipe fan through a one-entry `RoundFans`, the Bouncer gets no `ClusterFans`, and `excludesLenses` is never consulted.
  The recipe key alone then sets `ClusterExcludes` as today, and card 7's new subtest pins that the prompt still asks for `exclude_lenses`.
  The review row records `fans_recorded` and the recipe fan, so `excludedLenses` finds eligible records.
- Focus 3, one `way forward:` clause: `seatengine.startSeat` wraps the start error with `ModelKey.Wrap`, whose zero-key default case replaces the generic clause with exactly one, and `twillEngineRefusal` cuts at the last `; way forward: ` and appends its own step, so the final message carries one.
  The card's start-error rows drive a real `shuttleengine.Runner` through `seatengine.RunnerShuttle` over a `shuttlefake` engine with no `RequirementSupporter`, so the wrap runs, and each row asserts the clause count is one.
- Focus 4, live-fixer probe before archive: card 5 keeps the rule.
  Without recorded names it probes under the single-fixer profile first and archives and refuses only when the probe finds no live fixer; its test row covers a flagless record with a live fixer resumed as the single fixer with nothing archived.
- Focus 5, stale `fix_lenses` and `fix_forks`: `fixedExit` returns from its gate-failed branch before the usage merge, so a gate-failed twill fix writes neither key, and the merge that follows the gate check writes them only for a result naming lenses.
  A round is re-run after a gate-failed fix with the usage record untouched by that fix, so a later single fixer leaves an empty `fix_fan` with no lenses or fork count.
