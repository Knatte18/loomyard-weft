---
verdict: BLOCKING
findings:
  - id: chair-include-breaks-single-agent-composer
    severity: BLOCKING
    class: design
    location: _lyx/plan/04-discussion-advisor-stencil.md
    summary: Card 4 adds the seat-directive-chair include to loom-template-discussion while loomengine.composePrompt still fills that stencil with no includes until card 7, so every DiscussionSpec-path test and the live row fail from card 4 on
  - id: seat-runner-gate-once-own-card
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-discussion-write-row-on-seats.md
    summary: The shedfake.SeatRunner gate-once contract is a cross-package testkit surface used by shedadapters, shedrecipe, loomrecipe and loomcli, bundled into card 6 instead of its own earlier card
  - id: seat-runner-gate-once-double-eval-and-blast
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-discussion-write-row-on-seats.md
    summary: The fake evaluates table.Gate after RunFn, but card 6's parent-review RunFn already runs the script over table.Gate, and existing gated-table tests such as TestMultiLLMEntry_ComposedTable are neither targeted nor named in the ImpactSummary
  - id: advisors-mapping-refusal-way-forward
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-loom-config-discussion-advisors.md
    summary: A mapping value for discussion_advisors is refused inside ModelSpecList.UnmarshalYAML with the review-list way forward demanding a non-empty list and no entry index, so card 3's mapping row cannot pass as planned and the message contradicts empty-legal
  - id: d4-deviations-unrecorded
    severity: MEDIUM
    class: decision
    location: _lyx/plan/00-overview.md
    summary: Shared Decisions absent-key-takes-template and resolve-in-the-builder override D4 and two acceptance criteria (absent key loads as zero advisors; an unresolvable entry is refused at load) without a decision-record entry
  - id: seatengine-resigns-vs-additive-constraint
    severity: MEDIUM
    class: decision
    location: _lyx/plan/01-seat-table-optional-values.md
    summary: Card 1 re-signs the shipped composePrompt and validateValues and card 2 replaces MultiLLMProducer's table field, while the Constraints require every change to those packages to be additive and never a signature change of what ships
  - id: stale-discussionspec-references
    severity: LOW
    class: scope
    location: _lyx/plan/07-retire-discussion-spec.md
    summary: After card 7, comments in loomengine/review.go, loomengine/plan.go, shedadapters/singlellm.go, shedbuild/build_engines_test.go and loomshed/discussionwrite.go still cite DiscussionSpec or a wrapped SingleLLMProducer; only mergeresolve is corrected
  - id: reconcile-test-duplicates-mechanism
    severity: LOW
    class: design
    location: _lyx/plan/03-loom-config-discussion-advisors.md
    summary: TestConfig_DiscussionAdvisorsKeptThroughReconcile is a new top-level test for behaviour that splits into yamlengine's TestReconcile (open map carried whole) and configreg's TestLookup (membership), which take rows instead
---

### [BLOCKING] Card 4's chair include breaks the single-agent composer for cards 4 to 6

**Location:** `_lyx/plan/04-discussion-advisor-stencil.md` (the chair-stencil half), with `_lyx/plan/05-discussion-table-builder.md` and `_lyx/plan/07-retire-discussion-spec.md`.

**Issue:** Card 4 adds `{{template "seat-directive-chair"}}` to `contracts/stencils/loom/loom-template-discussion.md`.
Until card 7, that stencil is still rendered by `internal/loomengine/prompt.go`'s `composePrompt`, which calls `stencil.FillOptional`, which is `FillWith(template, nil, ...)`.
`internal/stencil/stencil.go`'s `attachIncludes` refuses any include action absent from the includes map: `stencil: include %q has no entry in the includes`.
The included block also declares the required markers `advisor_names`, `failed_advisors`, `inputs` and `output_files`, which `composePrompt` never fills.
So from card 4's commit on, `TestDiscussionSpec`, `TestDiscussionSpec_ReadsStencilAtCallTime`, `TestDiscussionSpec_PatternDirective`, `TestComposePrompt_RendersMarkers`, the discussion rows of `TestCompose_FrictionDirective` and `TestProducerSpecs_SkillsAndParentDirective`, and `loomcli`'s `TestWire_DiscussionSpecEvaluatesToExpectedShape` all fail.
Card 6 does not fix this either, since it leaves `DiscussionSpec` and its tests in place for card 7.
The production Discussion-Write row cannot compose its prompt between cards 4 and 6.
This breaks the stencil's "builds on its own" and "independently committable" card rules, and each card's own gate fails.
There is a second ordering knot: card 5's `TestDiscussionTable` asserts that the chair prompt names every advisor strand, and only the include's `{{.advisor_names}}` renders those names.
So that assertion needs the include, but the include cannot land while the old composer is live.

**Fix:** Keep card 4's advisor stencil, its registration, the `discussion-advisor` role entry and the chair stencil's prose changes.
Move the single include line `{{template "seat-directive-chair"}}` (and the chair-stencil claims that depend on it) into card 7, the card that deletes `composePrompt` and rewrites its tests.
Move card 5's assertion that the chair prompt names every advisor strand into card 7 too, as a row of `TestDiscussionTable` that card 7 then edits.
Alternatively, fold card 7's deletions into card 6, so that the composer goes in the same card that switches the row, and place the include there.

### [MEDIUM] The SeatRunner gate-once change belongs on its own card

**Location:** `_lyx/plan/06-discussion-write-row-on-seats.md`, the `internal/testkit/shedfake#SeatRunner`, `SeatRunner.Run`, `SeatRunner.Resume` and `TestSeatRunner_LastEntryRepeatsAndFnsOverride` targets.

**Issue:** Card 6 is held together by the `Env.DiscussionSpec` → `Env.DiscussionTable` swap, which forces every filler into one card.
That justifies the shedrecipe, envkit, loomrecipe, loomcli and battencli edits.
It does not justify the fake's new gate-once contract, which compiles and tests on its own and is reused across packages: `shedadapters/multillm_test.go`, `shedrecipe/entries_multillm_test.go`, `shedrecipe/multillm_fixture_test.go`, `shedrecipe/fixture_test.go` and `envkit` all construct it.
The rubric's granularity rule gives a symbol reused across packages its own card, even when one card is its only consumer.
As the plan stands, a reviewer of card 6 has to judge a testkit contract change inside the row switch.

**Fix:** Split the `shedfake.SeatRunner` gate-once change, with its `TestSeatRunner_LastEntryRepeatsAndFnsOverride` rows, into a new card before card 6, renumbering the later cards.
That card targets every existing consumer test the new contract changes (see the next finding).

### [MEDIUM] The fake's gate-once contract double-evaluates in the parent-review test and changes untargeted gated tests

**Location:** `_lyx/plan/06-discussion-write-row-on-seats.md`, the `SeatRunner` paragraph and the `parentreview_integration_test.go` paragraph.

**Issue:** The card says that `Run` and `Resume`, after answering `RunFn`, `ResumeFn` or the scripted entry, evaluate `table.Gate` once on a done chair.
`shedfake.Shuttle.RunGated` avoids double evaluation because its `RunFn` receives only the spec.
`SeatRunner.RunFn` receives the table, gate included.
The card's own `parentreview_integration_test.go` rewrite has a `RunFn` that "runs the test's script over `table.Gate` and returns the chair's result".
That script already evaluates the gate list through `arrive`, and it ends in a done chair, so the fake then evaluates the parent-review gate a second time.
The card promises to keep every assertion of that exchange, but the extra gate call is unaccounted for.
Separately, the contract changes every existing consumer whose table carries a gate.
`TestMultiLLMEntry_ComposedTable` in `internal/shedrecipe/entries_multillm_test.go` builds a table with a `plan` gate and scripts a done chair.
That test is not a card-6 target, and the ImpactSummary does not mention it.

**Fix:** State in the contract that the fake skips its own gate evaluation when `RunFn` or `ResumeFn` is set, or when the answered result already carries `Gate`.
Under either rule, the parent-review script stays the only evaluator.
Add `internal/shedrecipe/entries_multillm_test.go` (and any other gated-table consumer) to the card that carries the fake change, and name the effect in its ImpactSummary.

### [MEDIUM] A mapping value's refusal cannot carry the advisor way forward as planned

**Location:** `_lyx/plan/03-loom-config-discussion-advisors.md`, the `TestLoadConfig_Refuses` rows and the refusal-spec row.

**Issue:** `discussion_advisors` is typed `ModelSpecList`.
A mapping value is refused inside `ModelSpecList.UnmarshalYAML` as `modelSpecShapeError`, before `validateAdvisorSpecs` runs.
`LoadConfig` wraps that error as `loom config key %q: value at line N is neither ...; set the key to a model-spec, or to a non-empty list of model-specs`.
That message has no entry index.
It also tells the operator that the list must be non-empty, which contradicts D4's empty-legal rule.
Card 3 plans a mapping row "asserting the key, `entry 1` or `entry 2` and the way forward", meaning the advisor way forward.
Its refusal-spec row also gives that trigger "the message's" way forward.
Neither `ModelSpecList.UnmarshalYAML` nor `modelSpecShapeError` is a target, so the row as planned cannot pass.

**Fix:** In `LoadConfig` (already a target), when the shape error's key is `discussion_advisors`, return the advisor way forward instead of `modelSpecListWayForward`.
The mapping row then asserts the key and the advisor way forward, with no entry index, and the refusal-spec row's way forward matches.

### [MEDIUM] Two Shared Decisions override D4 without a decision-record entry

**Location:** `_lyx/plan/00-overview.md`, `### Decision: absent-key-takes-template` and `### Decision: resolve-in-the-builder`, carried into cards 3 and 5.

**Issue:** D4 says the load normalises "an absent key" to the empty list.
D4 also says every entry is "parsed and resolved at load", and the acceptance criteria repeat both points: an absent key loads as zero advisors, and an entry that fails to resolve is refused at config load.
The plan reverses both.
An absent key takes the template's one advisor, and card 3 adds a `TestLoadConfig_Loads` row pinning exactly that.
Registry resolution moves to `DiscussionTable` at the Discussion-Write call, so an unresolvable entry halts the run there instead of at load.
Both reasons hold against the code: `configengine.load` fills a present file's missing keys from the template, and `LoadConfig` has no registry.
Still, each one changes an answer-key outcome.
The plan stencil routes such a change through `lyx loom decision add` before planning on it, and the record's `Added after Discussion` section holds no such entry.

**Fix:** Record each deviation with `lyx loom decision add --by parent`, stating the template-fill and no-registry reasons, so the answer key and the plan agree.
Otherwise, bring cards 3 and 5 back to the record's outcomes.

### [MEDIUM] Card 1 and card 2 change shipped signatures under an additive-only constraint

**Location:** `_lyx/plan/01-seat-table-optional-values.md` (`validateValues`, `composePrompt` re-sign arrows) and `_lyx/plan/02-multillm-table-source-and-preparation.md` (the `table` field becomes `tables`).

**Issue:** The Constraints state that every change here to `MultiLLMProducer` and the seat engine "is additive (a new constructor, new fields), never a signature change of what ships", because `burler-multi-llm` edits both packages in parallel.
Card 1 re-signs two shipped functions and still calls itself additive.
Card 2 removes the shipped `table` field in favour of `tables`.
These changes are unexported, but they edit exactly the lines a parallel branch is likely to touch, which is the merge risk the constraint exists to prevent.
The plan never settles whether unexported members fall under "what ships".

**Fix:** Keep the shipped signatures.
`Table.Validate` can pass `validateValues` a copy of `t.Values` with the blank listed names removed.
`startSeat` and `Prompts` can call a new composer that takes the optional list, while `composePrompt` stays as it is.
`MultiLLMProducer` can keep `table` and add `tables` and `prepareFreshSpawn` beside it.
Otherwise, record a decision that the constraint covers exported members only, and drop "It is additive" from card 1.

### [LOW] References to the retired single-agent path survive card 7

**Location:** `_lyx/plan/07-retire-discussion-spec.md` and `_lyx/plan/06-discussion-write-row-on-seats.md`.

**Issue:** Card 7 corrects only `internal/mergeresolve/spec.go`.
Several comments still cite the deleted `loomengine.DiscussionSpec` as a live sibling or shape:

- `internal/loomengine/review.go:3`
- `internal/loomengine/plan.go:14`
- `internal/shedadapters/singlellm.go:62`
- `internal/shedbuild/build_engines_test.go:26`

After card 6, `internal/loomshed/discussionwrite.go:17` and `:39` still say the wrapped producer is a `*SingleLLMProducer`.
None of these is a target.

**Fix:** Add the four files' owning members, or the files themselves, to card 7's `**Edit:**` with a sentence to re-point each comment.
Add `internal/loomshed#discussionWrite`, or `internal/loomshed#NewDiscussionWrite`'s doc, to card 6.

### [LOW] Card 3's reconcile test is a new top-level test for split-covered behaviour

**Location:** `_lyx/plan/03-loom-config-discussion-advisors.md`, `plan:internal/loomengine#TestConfig_DiscussionAdvisorsKeptThroughReconcile`.

**Issue:** The behaviour has two parts.
The first is that a key listed in `ConfigOpenMaps` is carried whole through reconcile, which belongs to `internal/yamlengine`'s `TestReconcile` ("open map carries a list whole").
The second is that `discussion_advisors` is in the loom open-map list, which card 3 already adds to `configreg`'s `TestLookup`.
Loading the merged bytes is covered by the `TestLoadConfig_Loads` rows card 3 adds.
`TestReconcile` has no scalar or null open-map row, and that gap is real, but it belongs to the mechanism's test, not to a new loomengine top-level test.

**Fix:** Add `TestReconcile` rows for an open-map key held as a scalar, as `""` and as null, and drop the new loomengine test.
Card 3 then targets `internal/yamlengine/reconcile_test.go`, and the `internal/loomengine` budget row rises by zero in card 3, with card 7's final count adjusted.

## Focus points settled with no finding

- Card 5's added `mode_rules` sentence is licensed by D6's paragraph on the autonomous `mode_rules` text, and card 6's gate-once change is a consequence of D1's row switch; the scope findings above concern only its shape and placement.
- Card order: card 6 depends on nothing in cards 7 or 8.
  Card 5's `Prompts`, stencil and config uses all come from earlier cards.
  Card 7 deletes nothing that card 6 still references.
  The one ordering break is the include in the first finding.
- Re-sign arrows: `MultiLLMProducer.Call`, `loomCLI.wire`, `engineMinimalConfig`, `SeatRunner.Run` and `NewMultiLLMProducer` keep their signatures, so they take no arrow.
  Card 6's `newLoomShuttle` and `runLoomShuttle` re-signs carry arrows.
- Attack surface: card 1 bounds the optional list (only listed table values may be blank, and reserved names are refused).
  Card 2 bounds the preparation, card 6 states the nil-preparation rule, and card 3's validation is a refusal with a way forward.
- Verify: `./...` covers every targeted package, `internal/testkit/...` and `cmd/lyx` included, and the section follows the stencil's order.
- Test budget: the `internal/loomengine` row moves +1 (card 3), +2 (card 5) and then to the post-deletion count (card 7), matching the five deletions.
  Card 6's create-plus-delete in `loomcli` nets zero.
