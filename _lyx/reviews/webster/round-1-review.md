---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/planglyph/resign.go:100
    summary: resignAccepted and csharpResignWay pass glyph.CSharp to glyph.Parse outside internal/planparser, breaking the declared PATTERN-glyph-conversion-chokepoint text and Shared Decision 1
    fix: Add a helper in internal/planparser (for example ParseMintedID(plan, id) beside ParseRef in glyphref.go) that parses a quarry-minted C# id and is the only place that names glyph.CSharp for it. Replace both glyph.Parse(glyph.CSharp, res.ID) calls in resign.go (lines 100 and 138) with it, and drop the glyph.CSharp parse from planglyph. Alternatively amend the declared PATTERN text and pattern/PATTERN-glyph-conversion-chokepoint.md to carve out quarry-minted ids, but then the plan's declared text must change too, so the helper is the fix.
    origin: lens:code-pattern-fit
  - id: F2
    severity: MEDIUM
    class: scope
    location: PATTERN.md:7
    summary: Card 14 edits two PATTERN entries the task did not declare, cwd-resolution and sparse-hub-base, and weakens both
    fix: Restore the original text of both entries in PATTERN.md. cwd-resolution regains "; nothing else calls `os.Getwd` or `git rev-parse`" and sparse-hub-base regains ", never a template fill". Only PATTERN-glyph-conversion-chokepoint may change in this diff.
    origin: lens:code-pattern-fit
  - id: F3
    severity: LOW
    class: design
    location: internal/planglyph/languagemismatch.go:46
    summary: A go plan with a .cs path target and an unresolved Go member glyph raises plan-language-mismatch on top of the member's own glyph-not-found, and the set-csharp advice misleads
    fix: In languageMismatchFindings, do not count a member glyph that resolved to no symbol toward nonPathTarget when deciding csharpPath && nonPathTarget. Only a non-path target whose answer holds a symbol or listing should count. Add a row to TestGlyphChain_CSharp for the unresolved Go member beside a .cs path.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/planglyph/csharphead.go:300
    summary: renameCSharpDecl accepts an instance constructor and rewrites it to the new name under the old owner, though a constructor is named by its type
    fix: Make renameCSharpDecl refuse a constructor head, whose declared name equals its owner's name, with the same rename-old-unresolved finding the operator and finalizer cases get, and add the constructor to the refusal list in renameCSharpHead's finding text. Add a row to TestRenameCSharpDecl. If quarry.Name already rejects the head with a clear error, instead pin that answer with a TestGlyphChain_CSharp row.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: tools/plandag/graph.go:69
    summary: buildCardGraph, dependencyKind and columnsOverlap now parse refs through ParseRef, so C# member glyphs overlap by owner prefix in the card graph, with no csharp test row
    fix: Add a csharp-plan row to the tools/plandag tests whose two cards name C# member glyphs under one owner, asserting the dependency or overlap the graph reports.
    origin: lens:code-tests
  - id: F6
    severity: NIT
    class: consistency
    location: internal/planparser/validate_test.go:2048
    summary: The doc comment on TestValidate_LanguageRecognized still says only go and none are accepted, though the table has a csharp accepted row
    fix: Update the doc comment to say go, csharp and none are accepted.
    origin: lens:code-affected-set
---

### [BLOCKING] Alphabet passed to glyph.Parse outside planparser

**Location:** internal/planglyph/resign.go:100 and :138
**Issue:** `resignAccepted` and `csharpResignWay` call `glyph.Parse(glyph.CSharp, res.ID)`.
The plan's first Shared Decision and the declared text of `PATTERN-glyph-conversion-chokepoint` say nothing outside `internal/planparser` passes a `glyph.Language` to a parse.
The ids are quarry-minted rather than plan refs, but the declared text has no such exception, and the rubric makes any violation of a declared entry BLOCKING.
The other `glyph.CSharp` uses outside planparser (`batcher/boundary.go:49`, `handle.go:133`, `handle.go:382`) compare a `Lang` or build a `Declaration`; none is a parse, so none is a finding.

### [MEDIUM] Undeclared PATTERN edits

**Location:** PATTERN.md:7 and PATTERN.md:14
**Issue:** Commit `a7c839e5c` (card 14) shortens `PATTERN-cwd-resolution` and `PATTERN-sparse-hub-base`.
Only `PATTERN-glyph-conversion-chokepoint` is declared, and an undeclared change is judged by the overview's current text, so these two entries lose binding wording without a declaration.

### [LOW] False plan-language-mismatch beside an unresolved Go member

**Location:** internal/planglyph/languagemismatch.go:46
**Issue:** `nonPathTarget` is set for any non-path target, including a member glyph that resolves to nothing.
With a `.cs` path target and no Go evidence, the plan is refused with advice to switch to `csharp`, which hides that the member glyph is simply not found.

### [LOW] Constructor rename accepted

**Location:** internal/planglyph/csharphead.go:300
**Issue:** An instance constructor's declared name token is its type name, so `renameCSharpDecl` yields a head such as `public Bar(int x)` under the old owner.
The doc lists only operators, indexers, finalizers and static constructors as refused.
How `quarry.Name` answers such a head was not confirmed, so check that before choosing between the two fixes.

### [LOW] No csharp row for the plandag graph

**Location:** tools/plandag/graph.go:69
**Issue:** The graph's ref parsing moved to `ParseRef`, which changes overlap for C# member glyphs, and the diff adds no test in `tools/plandag` for a csharp plan.

### [NIT] Stale test doc comment

**Location:** internal/planparser/validate_test.go:2048
**Issue:** The comment lists the accepted languages without `csharp`.

## Rejected

- Fingerprint restamp may be skipped after a crash between `RewriteRefs` and the restamp in `postBatchChecks` (code-correctness finding 3): speculative, the fork did not confirm that the two steps can be separated by a failure.
- Re-sign arrows checked before the resolve have no dedicated ordering test (code-tests): the C# rows in `TestGlyphChain_ResignScenarios` fail if the check moves after the resolve, so the behavior is pinned.
- `parseErr.Lang` read in `handle.go:382` as a chokepoint violation (code-affected-set): it reads a field of an error and passes no language to a parse.
- Glyph.Parse in `planparser/validate.go:556`: inside the planparser package, so allowed.
- Duplicate fixture rows in `TestGlyphChain_CSharp` (code-tests finding 3): the Edit, Create and Delete rows check different verdicts.
- No `Pattern-Departure:` trailers exist in the range, so no departure to judge.
