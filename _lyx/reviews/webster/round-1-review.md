---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/specs/loom-plan-spec.md:355
    summary: >-
      The rewritten deviation paragraph states the informational clause twice in one sentence and now says the union itself is informational.
    fix: >-
      In the first sentence of the paragraph under Deferred / forward-compat, drop the trailing clause that says the union is always informational, never blocking on its own, and end the sentence at the closing parenthesis with a plain predicate, for example: is what the guard compares the batch's changes against.
      Keep the Uses and pointer sentences as they are.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/planglyph/doc.go:97
    summary: >-
      The scope-outside-plan bullet lists a card's file or package target covering the symbols in its extent as if the check reported it, so the bullet no longer reads as a list of reported conditions.
    fix: >-
      Reword the bullet so the reported conditions stay two and the coverage is a parenthetical qualifier, for example: a symbol a completed batch's delta touched outside its own cards' targets (a file target covers every symbol in that file, a package target every symbol in a file directly in its directory), and a changed file the delta gives no symbols for that no card target covers, file-granular.
---

### [LOW] Spec paragraph repeats its informational clause

**Location:** contracts/specs/loom-plan-spec.md:355
**Issue:** The first sentence already carries the parenthetical "a file-list mismatch against this union is always informational, never blocking on its own" and then closes with "is always informational, never blocking on its own".
The subject of that closing clause is the union, but it is the mismatch that is informational.
The card asked to keep the two facts once each.

### [LOW] doc.go bullet reads as three reported conditions

**Location:** internal/planglyph/doc.go:97
**Issue:** The bullet lists what `ScopeGuard` reports.
The inserted phrase names coverage, which is a reason a symbol is not reported, in the same comma list as the two reported conditions.
A reader takes the bullet to say three kinds of finding exist.

## Checked and found sound

- `RefMember` has no leftover `refMember` reference; both `RefCovers` call sites were retargeted, the nil-plan default is tested in one `TestRefMember` row in the existing `targetdirs_test.go`, and the `internal/planparser` budget row rose by one.
- `symbolFilesCovered` skips member refs through planparser, maps the rest through `RefScope`, treats a directory scope as direct-children only, and returns false for an empty files list.
  Modified collects every `Before` and `After` file, Renamed consults both sides only when neither ID is in the union, and Created and Deleted use their own file; this matches card 2's Intent.
- The table in `TestScopeGuard_SymbolCoverage` carries every row the card lists and replaces the three deleted tests one for one; each row exercises a distinct path, and the planglyph and planparser packages pass under `lyx gate test`.
- The implementer body's step 2 rule matches card 3, the guard-step clause now points at it, and the bounded self-fix section and the sandbox rule are untouched.
