---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-review-amend-key.md:20 and _lyx/plan/15-loom-subject-wiring.md:9 and _lyx/plan/13-recipe-bouncer-subject.md:8 and _lyx/plan/19-burler-forkcount-retry.md:9 (target lists)
    summary: "the method that collects each card's forced edits still misses whole-struct test fixtures and enumerating package docs after a new field or key, so cards 7 and 15 would fail their own tests and cards 13 and 19 leave two docs stale"
    fix: "Add these targets, each validated together with `lyx loom validate-plan --overlay` (no new blocking finding), and say in each card's Intent what changes. Card 07, Edit list after `internal/configreg#TestModules_EveryTemplateKeyClassified`, add `internal/loomengine#templateConfig` and `internal/darnengine#templateConfig`, both gaining `ReviewAmendMaxBounces: 1`, the template default (loom's is the want every `TestLoadConfig_Loads` row compares with reflect.DeepEqual, darn's the base of every `TestLoadConfig` want). Card 15, Edit list after `internal/loomcli#TestWriteCarryOver_Real_CommitsTheRecordAlone`, add `internal/loomcli#TestWire_ReviewKeysReachTheEnv` and `internal/loomcli#TestWire_DarnRecipe`, whose expected SegmentSettings (wantSegments for the four loom segments, wantSegment for Darn-Review) gain `AmendMaxBounces: 1`. Card 13, Edit list after `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`, add `internal/shedrecipe/doc.go`, whose Segment settings paragraph names the amendment allowance and whose Bouncer row paragraph names the optional `subject` key and the seams it selects. Card 19, Edit list after `internal/burlerengine#TestEngine_Review_Outcomes`, add `internal/burlerengine/doc.go`, whose What the halves return paragraph gains `SessionStates` among the ReviewResult fields. To stop the gap recurring, the writer lists, for every field or key a card adds to a struct or template, each test that builds that struct whole (reflect.DeepEqual over a literal, slices.Equal over a key list) and each doc that enumerates its fields or keys."
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/02-shed-budget-episode.md:53
    summary: "card 2 lets a GenerationStart Stuck skip the budget comparison over a spent episode but its Intent names no bound on who sets the mark or how often"
    fix: "In the D6 paragraph of card 02's Intent, after the sentence ending with 'even over a spent episode', add a bound sentence: the mark is set only by a producer that starts a generation, which today is the Bouncer's successful seed of a new generation (card 12), once per generation and never on a re-bounce or degraded seed; Shed trusts the mark as it trusts BudgetExempt, and a generation starts only when the settled subject changed or Merge-Review's merge-in changed. No target change."
---

### [BLOCKING] Forced-edit enumeration still misses whole-struct fixtures and enumerating docs

**Location:** `_lyx/plan/07-review-amend-key.md:20`, `_lyx/plan/15-loom-subject-wiring.md:9`, `_lyx/plan/13-recipe-bouncer-subject.md:8`, `_lyx/plan/19-burler-forkcount-retry.md:9`

**Issue:** Round 1 raised this gap for card 7 and card 5 and the fixer closed those sites; the same method misses four more.
Each item below was read in the tree.

- Card 7 gives `review_amend_max_bounces` the template default 1 and a `ReviewAmendMaxBounces` field on both Configs.
  `internal/loomengine/config_test.go` defines `templateConfig()` (line 93), the whole Config the template loads to, and `TestLoadConfig_Loads` compares `LoadConfig`'s result with it by `reflect.DeepEqual` (lines 445 to 452 and 515 to 521).
  `internal/darnengine/config_test.go` defines `templateConfig(mutate)` (line 35), the base of the `want` in nearly every `TestLoadConfig` row.
  Both literals leave the field at 0 while the loaded Config carries 1, so each row fails; the card lists `TestLoadConfig_Refuses` and darn's `TestLoadConfig` but not the helpers whose literal must change.
- Card 15 makes `loomSegmentSettings` and `wireDarnReview` set `AmendMaxBounces` from that default.
  `TestWire_ReviewKeysReachTheEnv` (`internal/loomcli/wiring_test.go`, `wantSegments`, line 944) and `TestWire_DarnRecipe` (`internal/loomcli/wiring_darn_test.go`, `wantSegment`, line 164) seed the template, call `c.wire` and require `reflect.DeepEqual` over a literal that pins each `SegmentSettings` whole.
  Neither literal names the new field, so both fail, and neither test is among the card's targets.
- Card 13 adds `SegmentSettings.AmendMaxBounces`, a `subject` key on the Bouncer row and the Env seams it selects.
  `internal/shedrecipe/doc.go` ("Segment settings and row settings") lists what a segment carries and which optional keys the row reads, and is not targeted.
- Card 19 adds `ReviewResult.SessionStates`.
  `internal/burlerengine/doc.go` ("What the halves return") lists the fields `ReviewResult` carries, and is not targeted, while the same card targets the shuttle and shedadapters docs for the same change.

CLAUDE.md requires a changed package's doc in the same commit, and the decision record's Scope ends on the same rule.
The two test failures are BLOCKING because the commits would not pass their own tests.

### [LOW] Card 2's GenerationStart skip states no bound

**Location:** `_lyx/plan/02-shed-budget-episode.md:53`

**Issue:** The budget arm's condition gains `!output.GenerationStart`, so a Stuck carrying the mark is recorded as counted but never takes `budget_capped` and never halts, "even over a spent episode".
That is a new way past Shed's budget check, and the card's Intent says what it lets through but not what bounds it: which producer may set the mark, and how often.
The bound exists in card 12 (the seed Stuck of a new generation, once per generation) and in the decision record's D6, but the card that introduces the skip does not carry it, which the attack-surface question requires.

## Rejected

- Focus 2, settle and generation records written from the Bouncer: not a defect. Card 11 states that every write (`recordSettle`, `markSettled`, the `seedCall` and `approveWithoutReview` generation-record writes, `judgeCall`'s) runs only when the `Subject` seam is told and `liveSubject` answers true, with a test row for a seam answering false and one for a nil seam. Card 12's `startGeneration` is reached only from case 2 and case 3, which need a live subject to compare, so it writes nothing without the seam.
- Focus 3, the `KeepReviewRecords` carry after a failed carry: not a defect. In `internal/loomshed/prrework.go`, `archive` writes `record.json` last and `Call` then runs `recreateReviewRunDirs`; a failure there returns the error, and the next Call finds the round in flight (`roundFor`, identity equal), takes the `default` branch with no archive and reaches `recreateReviewRunDirs` again, which card 14 hooks. Card 14's test row names that retry, and `CarryRecords` skipping a target that holds a generation record keeps a repeat harmless.
- Focus 4, which time a PR-Gate reject is compared against with no settle record: not a defect. Card 12 compares it with the generation record's `JudgeAt`, which card 10 adds and card 11's `judgeCall` writes before every judge spawn, and card 12's last test rows pin both an older and a later reject.
- Focus 5, new admissions: cards 10 (patch-id merge-in read as unchanged), 15 (`harnessOwnFiles`), 16 (the verify-retry bypass) and 5 (`--by`) each state what they let through and what bounds it. Only card 2's `GenerationStart` skip does not (F2).
- Focus 6, refusal and way-forward texts: not a defect. Card 17's `loomBudgetWayForward` is identical in its Create glyph and its Intent; `lyx loom resume` and `lyx loom goto --to` exist; card 16's writer row resolves to `Discussion-Write`, `Plan-Write`, `Webster`, `Merge-In` and `Darn-Write` from `contracts/recipes/loom-recipe.yaml` and `darn-recipe.yaml`, and the refusal ends in a trailing `way forward:` clause that names no denied form.
- Considered and not raised: a Burler review or fix row keeping a spent count across an amendment after an interrupted settle. Their hand-backs are `BudgetExempt` (`internal/shedadapters/burlerreview.go`, `burlerfix.go`), so only the Bouncer's own count is the segment's budget and `GenerationStart` need only reset that row.
- Prior-round items re-checked against the plan as it stands, none regressed: round 1's F1 sites (`TestModules_EveryTemplateKeyClassified`, darnengine doc.go, `TestStatusCmd_EnvelopeKeySet`, the overview sentence), F2's gating of the Bouncer's record writes, F3's carry on every Call that reaches `recreateReviewRunDirs`, F4's patch-id bound, F5's `JudgeAt`, F6 to F9's texts. The fixer report disputed and deferred nothing.
