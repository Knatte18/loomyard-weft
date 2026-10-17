---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/05-wire-segments-and-setting.md:34
    summary: Card 5 puts the plan_reviewer_fan Setting read on a wiring whose seed holds no fan, and card 8 drops the one wiring that seeds own model keys
    fix: In card 5's `TestWire_ReviewKeysReachTheEnv` bullet, move the Setting assertion to the test's second wiring (the `fanned` variable, whose seed sets `plan_reviewer_fan` to `full`) and state the expected answer `"full"`, instead of `c.env.Setting("loom", "plan_reviewer_fan")` answering "the seeded value". In the same bullet say the `Segments` assertion covers both wirings, so `fanned.env.Segments` is asserted for its own-key seeds `discussion_reviewer_model` (Discussion-Review `ReviewKey` and `Models`) and `webster_fixer_model` (Webster-Review `FixKey` and `Models`). In card 8's `TestWire_ReviewKeysReachTheEnv` bullet name `fanned.env.Setting` as the kept read, and say only the `fanned` wiring's per-row fan and `RowModelKeys` expectations go while its `Segments` assertions stay.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/06-review-rows-read-segments.md:64
    summary: Card 6 says every lookup error names the template that lacks the key, but card 1's Load and decode errors name nothing
    fix: In card 6's Settings paragraph, replace "a lookup error keeps the lookup's own text, which names the template that lacks the key" with "a lookup error keeps the lookup's own text"; add that card 1's unregistered-module and unknown-key errors carry their own `way forward`, and that an error propagated from `configengine.Load` or the snapshot decode is a transient refusal that stays as it is, because for `loom` and `darn` the wiring's strict load of the same stack has already passed before the recipe builds. Keep the test sentence that `fakeSetting`'s text carries no way forward.
---

### [LOW] Card 5's Setting assertion names a wiring that seeds no fan, and card 8 would drop the only own-key coverage for two segments

**Location:** `_lyx/plan/05-wire-segments-and-setting.md:34` (and `_lyx/plan/08-retire-row-maps.md:42-43`)

**Issue:** Card 5 asserts "for its seeded `loom.yaml`" that `c.env.Setting("loom", "plan_reviewer_fan")` "answers the seeded value".
In `internal/loomcli/wiring_test.go`, `TestWire_ReviewKeysReachTheEnv` seeds `c` with `plan_reviewer_model`, `plan_judge`, `seed`, `webster_seed`, `plan_max_bounces`, `webster_max_bounces`, `webster_auto_accept`, `merge_max_bounces` and `plan_hard_max_bounces`, and no fan key.
Read literally on `c`, the answer is the template default `""`, which does not show that the lookup reads the pair's seeded file.
Only the `fanned` wiring in the same test seeds `plan_reviewer_fan: full`.
Card 8 repeats the `c.env.Setting` wording as "its own module's seeded fan key (`loom.plan_reviewer_fan`, from card 5)".
Card 8 also drops "the per-row map assertions card 5 duplicated in `Segments`", but card 5's `Segments` assertion is stated for the first wiring only.
The `fanned` wiring is the only one that seeds `discussion_reviewer_model` and `webster_fixer_model`, so once its per-row `RowReviewModels` and `RowModelKeys` expectations go, no assertion would pin the Discussion-Review reviewer key or the Webster-Review fixer key and models when they come from a segment's own key.
In the first wiring every fixer key is the shared `fixer_model`, so a `loomSegmentSettings` row reading the wrong perch's `...Keys.Fix` would pass.

### [LOW] Card 6 describes every lookup error as naming the template that lacks the key

**Location:** `_lyx/plan/06-review-rows-read-segments.md:64`

**Issue:** Card 6 says that "a lookup error keeps the lookup's own text, which names the template that lacks the key".
Card 1 specifies four lookup errors, and only one of them does that.
The unregistered-module error names `Names()`, the unknown-key error names the template, and the `configengine.Load` and decode errors that step 3 wraps with the module name carry no way forward, as card 1's own test paragraph says.
`PATTERN-refusal-way-forward` binds `internal/loom*` verbs, and `loomrecipe.New` is reached through them.
The refusal-spec exempts a raw I/O failure and a fail-loud wiring guard, and a `Load` refusal for invalid YAML or a guard key in the local layer is neither, so card 6's blanket sentence both misdescribes card 1 and leaves the exemption it relies on unstated.
The refusal is unreachable for the shipped recipes after `loomcli`'s wiring has loaded `loom.yaml` or `darn.yaml` strictly, which is the reason the sentence should give.

## Rejected

- Focus 1, the `PATTERN.md` bound, not a defect: `HeadroomBytes` is `16384*95/100` = 15564 in integer arithmetic, `PATTERN.md` is 15560 bytes, and the declared line is 268 bytes with its newline, so insertion reaches 15828 and the card's cut of at least 264 bytes lands at or under 15564.
  `pattern.Headroom` fails only when the head is longer than the base and longer than the line, so landing at exactly 15564 passes.
  The line in card 6 matches the declared line character for character, and it is placed after `PATTERN-shed-recipe-registry`, which exists at that position.
  Card 1 changes only `pattern/PATTERN-config-strictness.md`, so no other card grows the overview.
- Focus 2, card 1's assertions against card 1's own messages, not a defect: the unregistered-module and unknown-key rows assert text that card 1's messages carry (module, key, `way forward`).
  The guard-key row's expected text holds, because `configengine.Layers` wraps `ValidateLocal`'s `key "plan_auto_accept" is guard and cannot be set in a local override` and step 3 adds the module name.
  `plan_auto_accept` is a `CategoryGuard` key and `review_max_bounces` a `CategoryNone` key, per `loomengine.ConfigSpec` and `reviewbudget.Categories`.
- Focus 2, `Env` comment rewrite, not a defect: the phrase "Env carries roots and run-wide values only" that card 3 says to drop `only` from is the text at `internal/shedrecipe/recipe.go` line 119 (the shared `ReviewModels` and `ReviewTimeout` comment), and card 8 deletes that comment with its field while keeping `ReviewTimeout` standalone.
  `Env`'s opening comment is the other rewritten site, and the `Slug` comment's "roots and run-wide values" has no `only` and stays true.
- Focus 2, `contracts/specs/shed-recipe-spec.md`, not a defect: the file still holds exactly one statement of the rule, at line 42 starting "The rule that makes the `Env`-versus-`Config` split decidable", which is the sentence card 3 quotes and rewrites.
  No other contract or doc file restates it, and the old per-row field names appear in only one doc, `docs/research/haiku-judge.md`, which card 8 covers.
- Focus 2, `loomSegmentPerches` comment, not a defect: its claim that a new loom perch also adds a row to `loomPerches`, "which the hard-cap lookup keeps", matches card 8's keep of `loomPerches` for `perchHardCap` in `internal/loomcli/circling.go`, and card 7's refusal wording "the loom wiring's segment-to-perch table" matches card 5's naming of `loomSegmentPerches`.
- `validate-plan --overlay '{}'` reports `ok: true` for the plan as it stands, and `lens-review.md` is the artifact the `--lens-review` check reads, so it is not a stray file.
