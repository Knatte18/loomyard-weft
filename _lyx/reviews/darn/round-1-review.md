---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/reportengine/landings_test.go:237
    summary: No row pins that a trailer above the pair-opening commit still maps through the exact join, and none puts another slug's opening commit in a tag's history
    fix: In the existing exact-join row at landings_test.go:237 (or :258), add w.pairOpening for the tag's own slug below its trailer commit, and one w.pairOpening for a different slug further up the history. Keep the row's expected exact-join mapping unchanged. This reuses the row and adds no test, and it covers the decision record's second acceptance criterion and the exact slug match at landings.go:164.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/reportengine/landings_test.go:258
    summary: The walk's floor is never reached on a page after the first, so the growing-page path with an opening commit is untested
    fix: In the existing row at landings_test.go:258, which already pushes its trailer past firstTrailerPage, add w.pairOpening for its slug below the trailer commit, so the opening commit lands on a later page and the mapping stays exact.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/fabricengine/add.go:423
    summary: Nothing asserts the subject Add commits, and the report's tests build subjects through the same export, so a changed prefix would pass every test while the floor stopped matching real pairs
    fix: 'In the existing TestAdd origin-record test (origin_integration_test.go:52), assert that the records commit subject equals OriginRecordSubject(slug) and, once, that it equals the literal "fabric: record parent branch for <slug>".'
    origin: lens:code-tests
---

### [MEDIUM] Exact join under a pair-opening commit is unpinned

**Location:** internal/reportengine/landings_test.go:237
**Issue:** The decision record requires that a tag whose trailer sits above its pair-opening commit still maps through the exact join.
No exact-join row calls `pairOpening`, and the new row has no trailer above its opening commits.
A walk that returned early on the opening commit would pass every row except the new one.
The `opened == slug` comparison at landings.go:164 is likewise unpinned, since no history holds another slug's opening commit.

### [LOW] Floor on a later page is unpinned

**Location:** internal/reportengine/landings_test.go:258
**Issue:** The new row's histories are shorter than the first page, so the page-doubling loop never meets an opening commit on a later page.

### [LOW] Add's commit subject is unasserted

**Location:** internal/fabricengine/add.go:423
**Issue:** The subject text is the contract between pair creation and the report's walk, and no test reads it from a real `Add`.

## Rejected

- A direct unit test of `OriginRecordSlug` and `OriginRecordSubject` (code-tests, finding 4 in part): it would test a helper outside the public surface; the walk rows and the `Add` assertion in F3 cover the behaviour.
- The code-correctness, code-affected-set and code-pattern-fit lenses returned no findings; my holistic read of the diff against the board entry, the decision record and the summary found the scope, the doc.go addition and the summary's claims consistent with the diff.
