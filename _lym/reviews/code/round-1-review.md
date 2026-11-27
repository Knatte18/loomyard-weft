---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/planparser/latercaller.go#isProductionFunctionDeclaration
    summary: The check refuses valid plans whose new function is a test helper the guard never scans, and the summary claims such a card stays silent.
    fix: "Make three changes. (1) In internal/planparser/latercaller.go, add one named constant listing the test-support package directories the guard roots (internal/testkit, internal/gitkit, internal/hubforge, each matching its directory and every directory below it). Make isProductionFunctionDeclaration return false for a handle whose unit lies under one of them, and add a row to TestValidateFormat_CreateCallerLater for a function created under internal/testkit that a later production-editing card names, expecting no finding. (2) In contracts/specs/loom-plan-checks-spec.md, under the create-caller-later entry, add that a helper of a package-internal _test.go file is declared on its package directory and so reads as production code. A later card that names it and changes production code raises the finding, and moving the Create to that card clears it. Name the test-support exemption beside the other skip conditions. (3) In _lym/landing/summary.md, rewrite the Design decisions bullet \"The check fires only when a later card can be the caller\" so it no longer claims a test-helper card stays silent when its later user also changes production code. Add the same limit under ## Limits."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/planparser/latercaller_test.go#TestValidateFormat_CreateCallerLater
    summary: Several classification branches have no row, so a mutant of each survives. They are the group-type filter and the handle-ref branches of refCanHoldProductionCaller.
    fix: "Add rows to TestValidateFormat_CreateCallerLater, each a creator card on card 1 and a later card 2 that Uses the handle and edits internal/dobby#Tick. Add builders for the Extends, Custom and Delete labels beside create and edit. Group-filter rows: the creator's Extends group lists internal/dobby#Tick (no finding); its Custom group lists internal/dobby/tick.go (no finding); its only other group is a Delete of internal/dobby#Old (finding). Handle-ref rows: the creator also creates a second func handle (finding); a const handle (finding); a test handle plan:internal/watch_test#helper with a func head (finding); and it edits plan:internal/other#Thing, a handle card 0 or no card creates (no finding). Each row asserts the exact Detail as the existing rows do."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/planparser/latercaller_test.go#TestValidateFormat_CreateCallerLater
    summary: Further behaviours the diff adds have no pinning row, and the finding's Ref field is never asserted.
    fix: "In TestValidateFormat_CreateCallerLater, add rows. The first is a creator of func ExampleLevel() with no testing. parameter, expecting no finding. The second has two later cards that can both call the function, expecting one finding that names the lower-numbered card. The third has a later card whose only production ref is the bare symbol dobby.Tick, expecting a finding. The fourth has a later card whose only refs are internal/dobby_test# and internal/watch_test#helper, expecting no finding. In the subtest loop, also collect finding.Ref and compare it with a want handle per row."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-spec.md:43
    summary: The Plan vs. schedule section still names only uses-later-target and delete-before-reference as the checks that refuse a dependency on a later card.
    fix: "In contracts/specs/loom-plan-spec.md, change the line \"`uses-later-target` and `delete-before-reference` refuse the cases a card's own refs show;\" to \"`uses-later-target`, `delete-before-reference` and `create-caller-later` refuse the cases a card's own refs show;\"."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/planparser/latercaller.go:46
    summary: A later card that creates the function's caller as a new func, and holds no other production ref, is skipped, so the creating card's failing gate goes undetected.
    fix: "In checkCreateCallerLater, judge the later card with a test that counts a func handle the later card itself creates as able to hold the call. Add a parameter to cardCanHoldProductionCaller, or a variant, that leaves the card's own func declarations out of the declared map for the caller side only. Keep the creator-side test unchanged. Add a row to TestValidateFormat_CreateCallerLater in which card 2 creates plan:internal/watch#Run with a func head and Uses the level handle, expecting the level finding naming card 2."
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:59
    summary: The Merge danger blast radius names planparser alone. The new refusal also reaches every plan gate and dispatch validation of a run in flight, whose plan may hold a pending card the check now refuses.
    fix: "In _lym/landing/summary.md, under ## Merge danger, change the Blast radius to name the plan gates (loomshed's ValidatePlan and the part check) and webster's dispatch validation, which run ValidateFormat. Add one sentence saying that a run in flight whose plan holds a pending card of this shape is refused at its next gate, until the plan moves the Create."
---

## Summary

The diff adds the plan-format check `create-caller-later` and wires it into `validate`.
It also adds a table test, two spec edits and one integration fixture moved from a func to a type.
It builds the board entry's goal by the plan-validation route, with the entry's first option, one card for the declaration and its first caller, as the check's way forward.
The summary gives reasons for leaving out the other two routes, which the entry offers as alternatives.
No finding is BLOCKING.

### [MEDIUM] The check refuses test helpers the guard never scans

**Location:** `internal/planparser/latercaller.go#isProductionFunctionDeclaration`

**Issue:** A handle names a package directory, never a file, so the plan text cannot tell a production function from a helper in a package-internal `_test.go` file.
In one case card 1 creates `plan:internal/watch#newFixture` -> `func newFixture() *Watcher` beside `internal/watch/fixture_test.go`.
Card 2 `Uses` it and edits `internal/watch#Run` and `internal/watch/run_test.go`.
That plan passes every card gate, yet card 1 is refused.
The same holds for a function under `internal/testkit`, `internal/gitkit` or `internal/hubforge`.
`cmd/lym/testonlycode_linux_test.go:35-39` roots each of those packages, so their functions never fail the guard.
The summary's Design decisions say that requiring a later card that changes production code keeps a test-helper card silent.
That holds only when the later card changes no production code.
The finding's first way forward, moving the `Create`, still clears it, so the plan is never stuck.

### [MEDIUM] The group filter and the handle-ref branches have no row

**Location:** `internal/planparser/latercaller_test.go#TestValidateFormat_CreateCallerLater`

**Issue:** The test builds only `Create` and `Edit` groups, so counting `Extends` and `Custom` and skipping `Delete`, `Rename`, `Move` and `Prosa` (`latercaller.go:96-101`) are unpinned.
The handle branches of `refCanHoldProductionCaller` (`latercaller.go:116-128`) have no row either.
Those branches are a test handle, a handle created on another card, and a `func` or `const` created on the same card.
The only handle refs besides the callee are the `type Watcher` and `var levels` rows.

### [LOW] Remaining unpinned behaviours

**Location:** `internal/planparser/latercaller_test.go#TestValidateFormat_CreateCallerLater`

**Issue:** No row covers any of these behaviours:
- the test-prefix skip without a `testing.` parameter;
- the `break` that names only the first later card;
- a bare symbol counted as a possible caller, which the spec states;
- a `_test` self glyph or test member as each condition alone.

`finding.Ref`, set at `latercaller.go:52`, is never asserted.

### [LOW] The Plan vs. schedule rule omits the new check

**Location:** `contracts/specs/loom-plan-spec.md:43`

**Issue:** The section says `uses-later-target` and `delete-before-reference` refuse the later-card dependencies a card's refs show, and that every other one is the writer's to avoid.
`create-caller-later` now refuses one more.

### [LOW] A later card that creates the caller as a new function is skipped

**Location:** `internal/planparser/latercaller.go:46`

**Issue:** Card 1 creates `A`, and card 2 creates `B`, which calls `A`, with `Uses: A` and no other production ref.
`refCanHoldProductionCaller` reads card 2's own `func` handle as unable to hold a call, so card 2 is skipped.
Card 1 is then silent, though it still fails its gate.
The skip is right for the creator side, where a new function is no caller, but wrong for the later card that writes the call.

### [LOW] The blast radius understates what the refusal reaches

**Location:** `_lym/landing/summary.md:59`

**Issue:** `ValidateFormat` runs at `loomshed.ValidatePlan`, at `checkPartPlan` and in `planglyph.dispatchFindings`.
A run in flight at merge time whose plan holds a pending creating card of this shape is refused at its next gate.
The summary's blast radius names `planparser` alone.
The `two-way` door is right: the diff changes no record or board format and no config meaning, and a revert removes the refusal.

## Focus verdicts

- **Done and resolved handles.** Not a defect.
  The finding sits on the creating card, and `dispatchFindings` drops every finding on a done card.
  Its neighbours' card-scoped findings get the same treatment.
  `CanonicalizeHandles` rewrites every occurrence of a handle across the card files, so the creator's declaration and a later card's `Uses` stay in step.
  `BindHandles` runs only after the creator's batch, when the creator is done.
- **Ref shapes.** Only the test-helper and test-support cases of F1 and the chain case of F5 are wrong.
  The other shapes are classified as the spec says: a file path, an extensionless path, a self glyph, a member glyph, a bare symbol, Move and Rename groups, and a handle created on this card or another.
- **Declaration heads.** Methods, generic heads and `func(` split correctly.
  `hasTestPrefix` and the `testing.` substring are broader than Go's test rule, which only means a missed finding (see Rejected).
- **Scope.** The reported case is caught when the later card names the handle.
  No registry or severity table needs the new id.
  Both stencils already state the writing rule, and the one stale spec sentence is F4.
- **Fixture.** `TestGlyphChain_RedundantPackage`'s case still creates a handle beside its package self glyph and expects `redundant`.
  It checks what its name states, and no assertion is loosened.
- **Claims.** The `Root-Cause:` trailer names the missing check, which the diff adds.
  The commit body and the spec's list of counted refs match the code.
  The summary's claims fail at F1 and F6.

## Rejected

- code-correctness F-3, the broad `hasTestPrefix` and `testing.` matches: they skip only rare names such as `Testable`, which costs a missed finding and never a wrong refusal. The spec states the rule as the code applies it.
- code-scope LOW, a later caller that does not name the handle: the plan text holds no other evidence of a caller, and the summary's Limits disclose it. No change to the diff can close it.
- code-tests MEDIUM items are kept as F2, and its four LOW items are merged into F3.
