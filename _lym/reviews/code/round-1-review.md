---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-template-master.md:238
    summary: The master stencil routes a plan_order refusal through fix-plan, but the plan fixer stencil sends any finding that reshapes a card to the plan writer, so each plan_order costs a fixer spawn that cannot fix it
    fix: In contracts/stencils/webster/webster-template-master.md, end the plan_order sentence at line 240 with a full stop instead of a colon, and state that a plan_order refusal skips the fix-plan rung, so the master writes outcome stuck to its outcome path with a stuck_reason quoting the refusal verbatim. Keep the plan_defect and tree_moved handling of the numbered fix-plan steps unchanged.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-template-master.md:265
    summary: The stencil quotes the later card's begin-batch refusal as "edit the plan so the named cards match the tree" for a symbol a batch's cards delete that a later card still references, which is now the plan_order refusal with a different way forward
    fix: In contracts/stencils/webster/webster-template-master.md line 265, stop quoting one wording. Write that the later card's own begin-batch refuses it with a plan_conflict refusal that names the way forward, without quoting the way-forward text.
  - id: F3
    severity: LOW
    class: consistency
    origin: assist:assist
    location: internal/planglyph/deleteorder_test.go:240
    summary: Two planglyph test readers still compare Finding.Check to the literal "delete-before-reference" beside the new planindex.CheckDeleteBeforeReference
    fix: In internal/planglyph/deleteorder_test.go line 240 and internal/planglyph/planfixture_helper_test.go line 106, import internal/planindex and compare f.Check to planindex.CheckDeleteBeforeReference in place of the literal. Add no test.
---

### [MEDIUM] plan_order is routed to a plan fixer that hands it to the plan writer

**Location:** `contracts/stencils/webster/webster-template-master.md:238-242`; `contracts/stencils/webster/webster-template-plan-fix.md:9-11` and its "Decide" section.
**Issue:** The master stencil lists `plan_order` among the causes that run `lym webster fix-plan`, and its `plan_order` sentence ends in a colon that attaches the numbered fixer steps to it.
The plan fixer stencil defines a mechanical finding as a stale target name and sends any finding that changes what a card does to the plan writer.
Moving a delete to a later card reshapes the cards, so the fixer returns `verdict: plan_writer` every time and the run ends stuck.
The classification is right and the end state is the same, but each `plan_order` refusal first spends a cold fixer strand.
The board entry asks only that the conflict be named as a plan-ordering one, so this is a routing gap the new cause opens, not a request the diff failed.

### [MEDIUM] The stencil quotes the old way forward for the delete-before-reference case

**Location:** `contracts/stencils/webster/webster-template-master.md:264-265`
**Issue:** Line 264 names the case "a symbol this batch's cards delete that a later card still references", and line 265 says the later card's `begin-batch` refuses it naming "edit the plan so the named cards match the tree, run ...".
That case is exactly what `PlanConflictError.Error` now renders as "move each named delete to a card after the card that still references it, run ...", so the quoted wording is wrong for the one case it describes.

### [LOW] Test readers keep the check-name literal

**Location:** `internal/planglyph/deleteorder_test.go:240`; `internal/planglyph/planfixture_helper_test.go:106`
**Issue:** `PATTERN-code-conventions` asks for one named constant for a name that can go stale.
The diff adds `planindex.CheckDeleteBeforeReference` and moves every emitter and the engine classifier to it, but two test readers keep the literal.
The assistant filed this as BLOCKING.
I keep it at LOW: both lines are unchanged and outside the diff, the tests pass either way, and a rename of the constant's value fails them loudly, so nothing goes stale silently.

## Rejected

- Lens code-correctness, LOW at `internal/websterengine/beginbatch.go:503`: a `plan_order` refusal on a delete from an already-begun card. The lens could not confirm that `ValidateDispatch` reports any begun card's delete; `ValidateDispatch` checks the pending cards only, as `internal/planglyph/planglyph.go:124` states, so the case does not arise.
- Lens code-tests, a `plan_order` test through the conflict record and CLI envelope: the lens itself rejected it, and I agree. Both copy the cause generically and the existing `plan_defect` rows pin that copy.
- Answer key's second item, running the check at plan accept or rebaseline: the diff keeps the existing plan-gate check and documents in `planglyph/doc.go`, `websterengine/doc.go` and the checks spec why a mid-run reference is first seen at dispatch. The entry says "consider", and the prose matches the code, so this is no scope gap.
- Pattern fit: the diff declares no PATTERN change and carries no `Pattern-Departure` trailer, and I found no violated listed rule beyond F3.
