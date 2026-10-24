---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/loomshed/gates_test.go:700
    summary: no row pins that a removal is credited to the first pending card naming its ID, so crediting the last or every naming card passes every row
    fix: In the TestNewPlanGate table of internal/loomshed/gates_test.go add a row with planSection removesB and patternOverview and patternBase removableUnderLine, where cardFence is namesB plus a blank line plus fenceOf(longEntryA) and secondFence is namesB, expecting no pattern-card-headroom and no pattern-headroom finding. Crediting the removal to the second naming card, or to both, then fails the row because card one's growth is not offset.
    origin: lens:code-tests
---

### [LOW] First-naming-card crediting is unpinned

**Location:** internal/loomshed/gates_test.go:700
**Issue:** The decision record credits a `removes` entry to the first pending card that names its backticked ID, and `cardHeadroomFindings` does so with `slices.IndexFunc`. Every new removal row has at most one naming card, so a change to the last or to every naming card would pass all rows. The behavior the diff adds has no covering row for the "first" part of the rule.

## Rejected

- Lens code-tests finding 2 (NIT, row `RemovalNoCardNamesShrinksTheProjectionBeforeTheFirstCard` has one card, so before-first and in-first placement are indistinguishable): `ProjectOverview` applies fenced lines then removals, so the two placements are equivalent for a single card, and the row still pins that an unnamed removal is applied. A name nit, not a defect.
- Stencil change in `contracts/stencils/loom/loom-template-plan.md` outside the record's Scope: the two added lines tell plan writers about the new check and the backtick-naming rule the crediting depends on, drop or weaken no existing rule, and match the record's crediting decision. Not an unlicensed change.
- `internal/planparser` change outside the record's Scope: `PATTERN-sole-parsers` requires card text to be read through `planparser`, and the record's own constraint says so, so `CardNamesPatternEntry` is the licensed route. The `plan.go` comment edit keeps the `Card.Text` readers list accurate.
- Commit trailer names Opus 5.5 where the session attribution names Sonnet 5.5: not a PATTERN entry or a rubric item, and no `Pattern-Departure:` trailer is needed.

All other items the four lenses listed were judged correct: removal crediting, boundary and empty inputs, done cards never projected, the whole-plan `pattern-headroom` text unchanged, docs and spec matching the code, and every acceptance criterion covered by a row.
