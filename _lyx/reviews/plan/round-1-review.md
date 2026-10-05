---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/03-review-rubrics-test-economy.md
    summary: Card 3 adds a third bullet under Webster-Review's Also flag but leaves the rubric's intro sentence "The two dimensions under Also flag" untouched, so it goes stale
---

### [LOW] Webster-Review intro counts two dimensions under `## Also flag`

**Location:** `_lyx/plan/03-review-rubrics-test-economy.md`, the `loom-rubric-webster-review.md` paragraph; the sentence it leaves alone is `contracts/stencils/loom/loom-rubric-webster-review.md:13`.

**Issue:** The rubric states "The two dimensions under `## Also flag` are added on top of that base".
`## Also flag` holds two bullets today (comment-convention compliance, per-card mechanical check).
Card 3 adds a Test economy bullet as a third and edits nothing else in that file, so the sentence becomes false the moment the card lands.
It is also a perishable tally the rubric did not need.

**Fix:** Card 3 rewords that sentence to carry no count ("The dimensions under `## Also flag` …") and says so in its Intent.

## Checked, not defects

- Cards 2, 3 and 4 edit disjoint `wordingClaims` rows of `claims_test.go` (the `loom-template-plan.md` row; the plan-review and webster-review rows; the `loom-template-discussion.md` and `burler-step-3-fix.md` rows), so landing in card order cannot overlap.
- Card 3's Webster-Review case list matches D5's four cases one for one, and both new bullets state the named-covering-test bound.
- `refusal-spec.md`'s only test wording is the wiring-guard class's "no per-row test" and the wiring-guard row's "none per row; grouped", both of which D8 keeps, so the file already satisfies "no longer requires one test per refusal" and no card is needed.
- Every old phrase D-record lists for the spec, the plan stencil and the Plan-Review rubric appears verbatim in its file and is carried by a card edit.
- `go test ./...` runs `internal/planparser` and `contracts/stencils`; `go test -tags integration ./contracts/...` covers the touched packages' tagged tests.
  No test reads the spec's worked example, so the golden fixture's consistency rests on card 1 leaving the fenced bodies untouched, which it states.
