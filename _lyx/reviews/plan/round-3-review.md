---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/02-stencil-include-form.md:16
    summary: Card 2 hands a bare include the value map by rewriting its pipeline to dot, but dot is not the value map inside the with and range bodies IncludeNames walks, so an include there fails at execution; the rewrite should use the root variable $
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-stencil-include-form.md:19
    summary: Card 2 does not say IncludeNames strips the leading comment, while card 3's block headers spell the include action and card 4 runs IncludeNames over block bytes for the nested-include refusal, so an unstripped parse makes each block report an include of itself
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/09-shedadapters-multillm.md:33
    summary: Card 9 plans a test of each chair outcome's mapping and a separate N = 1 comparison against SingleLLMProducer over the same outcomes; the adapter maps the chair result the same way regardless of N, so the comparison belongs as rows of the one outcome-mapping table
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/01-shuttle-quiet-hold-send-lock.md:23
    summary: Card 1's send-lock test does not assert that the gate spends no attempt on the notice-versus-re-prompt collision, which the decision record's acceptance criterion names
---

The plan carries every decision D1 to D14 and every constraint in the decision record, the card order holds (each `plan:` handle a card uses is created by an earlier card), and the `## verify:` section runs the tiers in the template's order over `./...`.
Every package a card targets is covered by that section.
The four findings below are all LOW: none blocks a card's gate or contradicts the answer key, and each is a one-line correction to a card's Intent.

### [LOW] F1 — the include rewrite hands the block dot, not the value map

**Location:** `_lyx/plan/02-stencil-include-form.md:16` (with `IncludeNames`'s walk at line 19)
**Issue:** Card 2 says `FillWith` "sets each bare include action's pipeline to dot in the parsed main tree before executing, and the block renders from the same value map as the main template".
In `text/template`, dot is the value map only at the top level and inside `{{if}}`; inside `{{with .x}}` it is `.x`'s string, and inside `{{range}}` it is the element.
The same card has `IncludeNames` walk `{{if}}`, `{{with}}` and `{{range}}` bodies, so it explicitly admits an include inside `{{with}}` or `{{range}}`.
Such an include passes the pre-execution marker check, since the values are present, and then fails at execution with "can't evaluate field" on a string.
The failure is loud, so no blank field renders, but the card's stated rule is false in the scopes the card itself admits.
**Fix:** In card 2's Intent, rewrite each bare include action's pipeline to the root variable `$` instead of dot, so the block always receives the map `Execute` was given.
Add one row to the `FillWith` rows in `TestFill`: a bare include inside a `{{with}}` body renders its marker from the value map.

### [LOW] F2 — `IncludeNames` does not say it strips the leading comment

**Location:** `_lyx/plan/02-stencil-include-form.md:19`
**Issue:** Card 2 says `FillWith` strips the leading comment of the main template and of every include, but says only that `IncludeNames` "parses a template".
Card 3 (`_lyx/plan/03-seat-directive-stencils.md:14`) has each block's header comment spell `{{template "seat-directive-chair"}}` or `{{template "seat-directive-advisor"}}`.
Card 4 (`_lyx/plan/04-seatengine-table.md:36`) has `Table.Validate` run `stencil.IncludeNames` over each block's read bytes to refuse a nested include.
`text/template` parses actions inside an HTML comment.
If `IncludeNames` does not strip, the chair block reports an include of itself, and `Validate` refuses every table that includes the shipped block.
The standard header's `{{if}}/{{range}}` wording would also fail to parse.
`TopLevelMarkers` strips the leading comment for this reason (`internal/stencil/stencil.go`), but card 2 does not say `IncludeNames` follows it.
**Fix:** In card 2's Intent, state that `IncludeNames` strips the leading comment before parsing, as `TopLevelMarkers` does.
Add an `IncludeNames` row whose leading comment spells a `{{template}}` action and whose body has none, expecting no names.

### [LOW] F3 — the N = 1 comparison duplicates the outcome-mapping test

**Location:** `_lyx/plan/09-shedadapters-multillm.md:33`
**Issue:** `multillm_test.go` plans "each chair outcome's mapping" and, separately, "a one-chair table mapped through a table-driven comparison against `SingleLLMProducer` over the same shuttle results".
`mapOutcome` reads only the chair's result, so a one-chair and a chair-and-advisor table map identically.
The two tests then pin one behavior twice, which `PATTERN-test-economy` forbids.
The plan should pin the one-chair mapping once at the engine and once at the adapter, each as rows of an existing test.
Card 6 does this at the engine (`_lyx/plan/06-seatengine-run.md`, the chair-outcome rows run over both table shapes).
Card 9 does not say its comparison is rows of its outcome table.
**Fix:** In card 9's Intent, make the outcome mapping one table-driven test whose rows give each shuttle outcome's expected `SingleLLMProducer` mapping, from `SingleLLMProducer` driven through `shedfake.Shuttle`, and assert `MultiLLMProducer` matches it over a one-chair table.
Drop the separate comparison test and size the budget row to match.

### [LOW] F4 — the gate's unspent attempt is not asserted

**Location:** `_lyx/plan/01-shuttle-quiet-hold-send-lock.md:23`
**Issue:** One acceptance criterion in the decision record says an advisor notice racing the gate's re-prompt at the chair's gated Done boundary "never types at the same time as the gate's re-prompt or pending send …; whichever goes second waits for idle, and the gate spends no attempt on it".
Card 1's new `run_test.go` test asserts the two lines are whole and in sequence, each with its own idle wait and delivery check.
It does not assert the gate's attempt count, so the criterion's last clause is pinned by no card.
**Fix:** In card 1's Intent, add to the `Run.Send`-beside-gated-Wait case an assertion that the gate's recorded attempts equal those of the same run with no concurrent send.

## Focus directives

The round's three directives, each settled:

1. The new-field and new-row cards list every test they must edit, so this is not a defect.
   Card 10 lists `envkit.FullEnv`, battencli's `intentionallyNil` and `loomCLI.wire`.
   `NilSeams` reports a nil map or interface, so both `Seats` and `Models` need filling.
   It is called only from `internal/loomcli/wiring_seams_test.go`, `internal/battencli/wire_seams_test.go` and envkit's own test.
   Card 11 lists `minimalEntries`, `TestNames`, `coverageGuardAllowedUnreachableEngines` and shedbuild's `engineMinimalConfig`.
   The other range over `Names()`, `TestLookup`, needs no edit.
   No other `Env` literal is checked for seams.
2. The plan states one include form: the bare `{{template "name"}}`.
   Card 2 refuses the dotted form, card 3's headers use the bare form, and cards 5, 12 and 13 compose through `FillWith`.
   Card 2 states one handing rule, but the rule is wrong inside `with` and `range` bodies (F1), and the include scan is underspecified (F2).
3. Card 6 pins the one-chair mapping at the engine as rows of its chair-outcome test.
   Card 9's adapter-level comparison is planned as a separate test (F3).
