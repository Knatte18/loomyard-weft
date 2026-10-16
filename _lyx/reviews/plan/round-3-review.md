---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-production-fans.md:12-13
    summary: Card 1 says every new lens keeps a bench text ending in its own Your list sentence, but the three fixer lenses have no such text, so the lens-text ruling is not stated unambiguously
    fix: >-
      In _lyx/plan/01-production-fans.md, scope the sentence at lines 12-13 ("New lenses, each a procedure that starts its fork's final message with its own list. They keep the bench text, which ends in its own `Your list:` sentence and carries no emphasis line") to the seven reviewer lenses (`interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `affected-set`, `guard-reach`, `way-forward`).
      Add one sentence for the five fixer lenses saying that `fix-consistency`, `fix-claims` and `fix-way-forward` keep the twill-fixer bench's lens text as it is, with no list sentence and no emphasis line, since the fixer's fork boilerplate returns findings only;
      that `fix-affected-set` ends by listing each finding it left unchecked, as D8 requires;
      and that `fix-proposal` is a procedure with no list sentence.
      Keep the sentence that no new lens carries the emphasis line, with D4 as the reason.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-production-fans.md:33
    summary: Card 1 exempts the new lenses from the emphasis-suffix assertion but leaves the template header comment, which says lens prose is always emphasis, unchanged
    fix: >-
      In _lyx/plan/01-production-fans.md, add to the Intent, next to the sentence about the emphasis-suffix assertion at line 33, that the header comment of `internal/burlerengine/template.yaml` (the paragraph from "Lens prose is always phrased as EMPHASIS" onward, line 27) is edited to say the dormant lenses are emphasis prose and the procedure lenses of the four production fans end in their own list instead, while the no-"ignore" rule still holds for every lens.
      The file `internal/burlerengine/template.yaml` is already a target of the card, so no target changes.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/01-production-fans.md:31
    summary: The new TestResolveFan row repeats the fan-order assertion that TestConfigTemplate_DecodesAndIsSelfConsistent already makes through its wantOrders map
    fix: >-
      In _lyx/plan/01-production-fans.md, delete the target line `internal/burlerengine#TestResolveFan` under Edit and the sentence at line 31 that plans the row.
      Extend the sentence at line 32 so `TestConfigTemplate_DecodesAndIsSelfConsistent` also adds the four new fans to its `wantOrders` map, with the lens order the card lists under "New fans", which resolves each against a zero `Config` through `ResolveFan` and so fails on a missing fan, a misordered fan or an undefined lens.
      Checked with `lyx loom validate-plan --overlay`: the card with the TestResolveFan target and the row sentence removed reports no blocking finding beyond the baseline.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/08-loom-per-round-fan-keys.md:64
    summary: Card 8 changes the unknown-fan refusal wording but does not say that the two existing unknown-fan rows of TestLoadConfig_Refuses, which assert the old wording, are rewritten
    fix: >-
      In _lyx/plan/08-loom-per-round-fan-keys.md, extend the `TestLoadConfig_Refuses` sentence at line 64 to say that the existing "unknown discussion_fan" and "unknown plan_fan" rows keep their key, fan name and "known fans: " assertions but replace their `to empty to run the segment solo` assertion with `entry 1` and `set the entry to ""`, since a scalar value is entry 1 of the key and takes the new wording.
      The "set discussion_fan with a malformed burler.yaml" row keeps its assertions, because the card keeps the read-failure wording.
---

### [MEDIUM] Card 1's lens-text ruling overreaches for the fixer lenses

**Location:** `_lyx/plan/01-production-fans.md:12-13`, against `docs/research/burler-twill-fixer-bench-2026-10-16/setup/selfcheck-block.md` and the decision record's D4, D5 and D8.
**Issue:** The card carries the focus's lens-text ruling in two sentences: all twelve new lenses are "a procedure that starts its fork's final message with its own list", and they "keep the bench text, which ends in its own `Your list:` sentence and carries no emphasis line".
That holds for `interactions`, `existing-paths` and `bound-breaker`, whose bench texts in `setup/iter1/burler.yaml` end in `Your list:`, and it fits the reviewer fork rules, which tell a fork to start its final message with the list its lens asks for.
It does not hold for `fix-consistency`, `fix-claims` and `fix-way-forward`: the card names `selfcheck-block.md` as their source, where each lens is a one-sentence bullet with no list sentence, and the fixer's fork boilerplate there returns findings only ("return 'no findings' when you have none").
D5 and D6 demand a list per lens; D8 demands none for the fixer lenses, so the card extends D5's property to lenses D8 never gave it.
A writer of card 1 then has two readings: keep the bench bullets, which contradicts the card's first sentence, or append a list sentence, which departs from the bench text D8 starts from and adds unbenched text.
The ruling on the emphasis line itself is stated clearly and agrees with D4, since the emphasis line "Report anything else you notice too" would contradict D4's rule that a fork reports only what its procedure produces; the gap is in the list half.

### [LOW] The template's header comment still says lens prose is always emphasis

**Location:** `_lyx/plan/01-production-fans.md:33`, `internal/burlerengine/template.yaml:27`.
**Issue:** After card 1 the template holds twelve lenses that carry no emphasis line, and the card makes the template test skip them.
The header comment of the same file (line 27, "Lens prose is always phrased as EMPHASIS, never exclusion") and its `my-lens` example, which ends in the emphasis line, then describe a convention the template itself departs from.
The card edits `template.yaml` but does not say the comment follows.
Operators copy the example block for their own lenses, so the comment is where the departure has to be recorded.

### [LOW] TestResolveFan row duplicates an existing assertion

**Location:** `_lyx/plan/01-production-fans.md:31`.
**Issue:** The card adds to `TestResolveFan` "a row resolving each of the four names against a zero `Config` and asserting the lens names in exactly that order".
`TestConfigTemplate_DecodesAndIsSelfConsistent` (`internal/burlerengine/config_test.go`, the `wantOrders` loop) already calls `ResolveFan(Config{}, fanName)` for each fan in a map and compares each lens name by position, failing on an unknown fan, an undefined lens or a wrong order.
The card edits that test anyway for its count pins, so the four fans belong in `wantOrders`; a second test asserting the same order is a redundant row under `PATTERN-test-economy`.

### [LOW] Existing unknown-fan rows of TestLoadConfig_Refuses assert wording the card changes

**Location:** `_lyx/plan/08-loom-per-round-fan-keys.md:64`, `internal/loomengine/config_test.go:592-593`.
**Issue:** The card prescribes one refusal for every non-empty list entry, `loom config key %q entry %d: <ResolveFan error>; define the fan in burler.yaml, or set the entry to ""`, and a scalar is a one-entry list.
The existing rows "unknown discussion_fan" and "unknown plan_fan" assert `to empty to run the segment solo`, which that wording drops, so they fail unless rewritten.
The card lists `TestLoadConfig_Refuses` under Edit and says it replaces the two `fan_review` rows and adds new ones, but states nothing about these two rows, so the pins the changed wording needs are not stated.

## Rejected

- Tests pinning counts or literal lists, beyond the finding above: every other site is carried by a card that lists the test under Edit.
  `TestConfigTemplate_DecodesAndIsSelfConsistent`'s lens, fan and per-fan entry pins are card 1's, and the new totals follow from the card's own lists of new lenses and fans.
  `TestLookup`'s `OpenMaps` list and `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` list in `internal/configreg` are card 8's, the latter with its sorted order stated.
  The `fan_review` Config literals in `internal/loomengine/config_test.go` (`templateConfig`) and `review_test.go` (`TestResolveReview`) are card 8's, and `TestWire_ReviewKeysReachTheEnv` and the `fan_review` key assertion in `TestBurlerReviewEntry_ClusterFan` are card 8's and card 7's.
  `TestModules_LocalLayerRefusals`'s `plan_fan` row stays valid, since the key stays local-denied.
- Card 8's `fanShapeError` triggers: both have a named input that reaches them.
  A mapping value (`discussion_fix_fan` as a mapping) reaches the default branch of `FanList.UnmarshalYAML`, and a `plan_fan` list holding a mapping entry reaches its sequence branch, the same two paths the existing "mapping review value" and "mapping inside a fix list" rows take through `modelSpecShapeError`.
  No upstream check rejects either input before `yaml.Unmarshal` in `LoadConfig`, and the way-forward text is carried by `fanShapeError.Error`.
- The `fan_review` retirement through `RetiredKeys`: `internal/loomengine/config.go` declares none today, but `configengine.Spec` carries the field for other modules and the card's declaration is licensed by D3.
- The test budget: the cards add one top-level test (`TestBurlerReviewProducer_Call_RecordsRoundFans`), and `internal/shedadapters` has room under its row in `cmd/lyx/testdata/test-budget.yaml`.
