---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/02-vocabulary-name-check.md:34
    summary: The bare-token rule looks up the handle's unit in the owner set unchanged, so a test declared in an owner package's external test package (unit internal/fabricengine_test) is refused, though the scan admits it because its owner check is by directory.
    fix: In card 2's Intent (line 34), have the bare-token rule pass the unit's directory to fabricvocab.SkipsBareCheck, which is the unit with a trailing _test stripped from its last path segment (internal/fabricengine_test becomes internal/fabricengine), and state that this is the directory the scan's owner check reads. In TestCheckFabricVocabularyName's pass rows (line 47), add a Create handle plan:internal/fabricengine_test#Test<Side>… built from fabricvocab.SideTokens(). Add a refuse row for a side-token member in plan:internal/configsync_test, which still fails under configsync's identifier rule. In card 3's loom-plan-spec.md row 41 text, say that an external _test unit counts as its directory.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-vocabulary-name-check.md:31
    summary: Card 2 re-derives the owned-handle list from Card.Declarations and IsHandleRef-filtered Card.Pairs, but the exported planparser.OwnedHandles (internal/planparser/handle.go:85) already returns exactly that list in body order.
    fix: In card 2's Intent, have CheckFabricVocabularyName iterate planparser OwnedHandles(card) instead of reading Declarations and Pairs itself. Move internal/planparser#IsHandleRef out of card 2's Uses and add internal/planparser#OwnedHandles in its place.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/08-lens-review-file.md:37
    summary: Seeding lens-review.md in TestPRRework_FirstRoundMovesGenerationZero repeats a behavior the test already covers, since prRework.planMoves moves every plan-directory entry and the test already seeds and asserts the non-card amendments.md.
    fix: In card 8, drop the TestPRRework_FirstRoundMovesGenerationZero bullet from Tests and internal/loomshed#TestPRRework_FirstRoundMovesGenerationZero from Edit. Keep the Intent sentence saying PR-Rework's round archive needs no change, and cite the existing amendments.md assertion as the coverage.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/00-overview.md:28
    summary: The Shared Decision says the plan host's rejection rule is rendered exactly as the record states it, but card 9's planTwillRejectionRule rewrites D5's text from the writer to you and splits it into sentences.
    fix: Do one of these two. Either change the Shared Decision bullet at 00-overview.md:28 to say the rejection rule and spawn point carry D5's and D3's content in the prompt's second-person voice, while the severity rule and the per-fork line are verbatim. Or render planTwillRejectionRule in card 9 (09-plan-twill-directive.md:70) word for word as D5 writes it.
---

### [BLOCKING] fabric-vocabulary-name refuses owner-package external tests the scan admits

**Location:** `_lyx/plan/02-vocabulary-name-check.md:34`

**Issue:** Card 2 passes `unit` straight to `fabricvocab.SkipsBareCheck`.
An external test package's handle unit is spelled `<dir>_test`: this plan writes `plan:internal/planparser_test#…`, and `internal/planglyph`'s tests use `shapes_test#TestExternal`.
The scan's owner set is keyed by the file's directory (`fabricVocabularyFailures` uses `dir := filepath.Dir(rel)` in `internal/lyxcwd/enforcement_test.go`).
So the scan admits a `_test.go` file in `internal/fabricengine` declaring `package fabricengine_test`.
The new check sees unit `internal/fabricengine_test`, finds no owner row for it, and since `first` is `internal`, it refuses.
This contradicts D9 ("its package is outside the vocabulary owner set").
The case is real: `internal/fabricengine/cleanupremotewarp_integration_test.go` and its siblings declare test names carrying a side token, and owner dirs such as `internal/boardengine` and `internal/fabriccli` hold external test packages.
Because the check is blocking at both plan gates, `validate-plan` and dispatch, a correct plan adding such a test would be refused, and its way forward (rename the member) is a wrong instruction.
None of card 2's pass rows reaches the `_test` unit form.

### [LOW] Card 2 re-derives `OwnedHandles`

**Location:** `_lyx/plan/02-vocabulary-name-check.md:31`

**Issue:** `planparser.OwnedHandles(c Card)` (`internal/planparser/handle.go:85`) returns every `Create` declaration's handle plus every `Rename` pair's `New` side that `IsHandleRef` accepts.
That is exactly the name set card 2's Intent spells out by hand.
A second copy of that enumeration can drift from the one the binder uses.

### [LOW] Redundant PR-Rework seed in card 8

**Location:** `_lyx/plan/08-lens-review-file.md:37`

**Issue:** `prRework.planMoves` (`internal/loomshed/prrework.go:420`) archives every entry of the plan directory, with no per-name logic for card 8 to change.
`TestPRRework_FirstRoundMovesGenerationZero` already seeds `amendments.md`, a non-card file, and asserts that it lands under `prior-generation/plan/`.
A `lens-review.md` seed takes the same code path, so it covers no new behavior (`PATTERN-test-economy`).

### [NIT] Shared Decision overstates verbatim rendering

**Location:** `_lyx/plan/00-overview.md:28`

**Issue:** D5 states the rejection rule in the third person ("such a finding the writer cannot refute from code it has read is fixed").
Card 9's `planTwillRejectionRule` keeps that content but says "Such a finding you cannot refute from code you have read is fixed" and splits the clauses into sentences.
The severity rule and the per-fork line do match the record byte for byte.
The overview's claim of "exactly as the record states them" therefore holds for two of the four texts.

## Focus questions settled

- Fidelity: every decision, constraint and acceptance criterion maps to a card.
  D1 is card 6, D2, D3, D5 and D8's spawn side are card 9, D4 is card 4, D6 is card 7, D7 is card 8, D8's template is card 5, D9 is cards 2 and 3, and D10 is card 1.
  D11 is correctly left out of the cards, per the parent's recorded decision.
  The four lens texts in card 5 match D8 verbatim.
  Card 4 adds hostNeutralClaims rows, card 8 adds the lens-review-only rotation branch (forced by `PriorPlanBlock` refusing an empty list), and card 9 adds the shared `twillLenses`, which D8 licenses.
  None of the three is unlicensed scope.
  The record's "`internal/loomengine` … `doc.go`" has no file to land in, since the package doc lives in `status.go` and names no producer.
  Card 9's `plan.go` header follows the Discussion precedent in `discussion.go`, so this is not a defect.
- Attack surface: card 9's Intent states what fork authorization lets through and its bounds (fork model, read-only and no Agent by boilerplate, the 15-wait give-up, gates and Plan-Review).
  Card 7 adds no bypass, since it removes only `PlanReadyMarker(l)`, treats a missing marker as success and runs only on a fresh spawn.
  Neither is a defect.
- Reachability: each prescribed refusal has a named input and a named test with no upstream check rejecting it.
  These are card 4's missing-value rows, card 6's `sideways` row, card 7's `NilPreparePlanSpawn`, error and wire rows, card 8's lens-only rotation, card 9's two `TestPlanSpec_Refuses` rows, and card 3's dispatch-completed row.
  The one reachability gap is F1's untested `_test` unit.
- Test economy: no existing `twilldirective` test covers an empty host value, so `TestDirective_MissingValueNamesTheMarker` is new behavior.
  `TestIndex_AppendsFabricVocabularyName` covers the index wiring, including the dispatch drop, which no planglyph test reaches today.
  `TestCheckFabricVocabularyName` is one table.
  The budget rows add up: twilldirective 3 to 4, planparser 90 to 91, planglyph 85 to 86, loomengine 43 to 44, and shedrecipe 100 to 101 against a row of 101.
  The redundancy found is F3.
- Edit glyphs and arrows: the Edit cards name members where their Intent names symbols.
  `composePlanPrompt` carries its re-sign arrow.
  `Values.ReviewedFiles` and `Env.PreparePlanSpawn` are struct fields and take none.
  `Directive`, `twillBlocks`, `PlanSpec`, `LoadConfig` and `NewPlanDirRotator` keep their signatures.
- Card order: no card uses something a later card creates.
- Verify: `go test ./...` and `go test -tags integration ./...` cover every targeted package, `contracts/stencils`, `internal/loomengine`, `internal/loomcli` and `cmd/lyx` included.
  The repository's only nested module is testdata, and the section follows the template's order.
- Granularity: card 1 is one reviewable move, since the `lyxcwd` edits cannot be separated from the package they call.
  Card 9 is large but is one spawn-side feature with its tests and refusal rows.
