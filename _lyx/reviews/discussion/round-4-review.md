---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: Caller coverage accepts a caller file covered by an earlier card, which is unsound for a Rename Old side or a re-signed member whose callers must change in the card that changes the signature
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: The over-reporting bound says only methods and fields are ambiguous, but the reused same-package pattern for a package-level name is a line-textual whole-word match that also hits selectors, struct keys, comments and strings, and it is blocking
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:149
    summary: The plan-writer note calls the delete-before-reference call site a plan-gate site, but it sits in resolvePass shared with ValidateDispatch, and D5 says caller coverage runs beside it yet never at dispatch
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:20
    summary: Scope states the file-paths-only rubric finding for any file paths, while D6 narrows it to Go file paths of non-test files
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:137
    summary: No acceptance criterion pins that re-sign head answers stay out of canonicalization, which a leak would break for the same member re-signed on two cards
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:135
    summary: The Rename to-side criterion does not name its one discriminating direction, because the Old side already trips the pairing in the other direction
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:74
    summary: D5 takes the head's unit from Glyph.UnitPath, a glyph-to-path conversion, where the quarry.Name Declaration needs the glyph's unit
---

## Focus answers

1. D5 now places the head-to-member match inside `CanonicalizeHandles` (decision-record.md:74), not `resolvePass`.
   `TestChokepointCallSites_PinnedToTheirGuardedFunctions` maps `Name` to `CanonicalizeHandles` and matches any `quarry.Name` call outside it, so joining the existing batched call passes the pin.
   D5 states the head answer is compared with its bullet's glyph and discarded, outside the canonicalization maps, the collision check and the `RewriteRefs` rewrite.
   That is a requirement the current loop in `handle.go` does not meet on its own, since it feeds every `nameResults` entry into `canonicalOwners`, and no test pin sees a leak; F5 asks for the observable criterion.
2. Scope (line 20), D6 (lines 93-94) and the acceptance criterion (line 142) name the same two rubric findings.
   Only the first one's qualifier differs between Scope and D6; see F4.
3. D3's Rename rule and the draft-spelling rule differ in one direction only, and the acceptance criterion requires it only through the words "exactly when"; see F6.

### [MEDIUM] Caller coverage accepts earlier-card coverage for Rename and re-sign

**Location:** `_lyx/discussion/decision-record.md:81`, Q6 in `support-log.md`.
**Issue:** The coverage rule is "a target of the same card or an earlier card".
For a `Delete`, an earlier card that removes the caller's reference is a legitimate order.
For a `Rename` Old side or a re-signed `Edit` member the callers have to change together with the declaration: an earlier card cannot write a call against a name or signature that does not exist yet, and D1 requires every card to leave the suite green.
A caller file that an earlier card touches for an unrelated reason satisfies the check, yet the rename or re-sign card breaks that caller.
The Bound for under-reporting (line 85) lists alias imports and files an earlier card creates, not this case, and Q6's rationale (card order, partition unknown) argues only the delete direction.
**Fix:** State the direction per kind: `Delete` coverage by the same or an earlier card, `Rename` Old side and re-sign coverage by the same card only, or state in the Bound why earlier-card coverage is acceptable for them.

### [MEDIUM] Over-reporting bound understates the package-level false-positive surface

**Location:** `_lyx/discussion/decision-record.md:83-84`.
**Issue:** D5 reuses `deleteorder.go`'s per-symbol patterns.
For a package-level member that file builds the same-package pattern as `\bName\b` applied to every line, so a file of the package holding `x.Name`, a struct literal key `Name:`, a method `Name`, a comment or a string containing the word matches.
The Bound says the name scan is ambiguous only for a method or field and blocking "only for package-level members", which reads as if the package-level match were reliable.
The Open risks section names only a shared local identifier.
Generic names (`New`, `Load`, `Run`, `String`) make the other cases routine, each a blocking finding at both plan gates whose way forward is to list a file the card does not change.
**Fix:** Either state in the Bound that a package-level same-package match is textual and over-reports on selectors, keys, comments and strings, and that listing the file clears it, or require the new walk to tighten the pattern (skip comments and strings, skip a name preceded by `.` for a package-level member).

### [LOW] Plan-writer note and D5 misplace the delete-before-reference call site

**Location:** `_lyx/discussion/decision-record.md:149` and line 81.
**Issue:** The note says the pattern reuse has "its plan-gate call site" in `planglyph.go`.
`LaterDeleteReferences` is called inside `resolvePass`, which `ValidateDispatch` shares, so `delete-before-reference` runs at dispatch too.
D5 says caller coverage "runs at the plan gates beside `delete-before-reference`" and also "never in `ValidateDispatch`"; a plan writer who follows the note and the word "beside" lands the new check in `resolvePass`, which D3 and D5 forbid.
**Fix:** Say the call site is in `resolvePass`, shared with dispatch, and that caller coverage goes in the separate plan-gate-only pass, not beside it.

### [LOW] Scope and D6 disagree on the file-paths-only rubric finding

**Location:** `_lyx/discussion/decision-record.md:20` against line 93.
**Issue:** Scope says "an `Edit` card holding only file paths while its Intent names specific symbols".
D6 says "only Go file paths of non-test files".
D6 also lets the stencil allow test files and non-Go files as paths, so Scope's wording would have the rubric flag a card the stencil permits.
**Fix:** Use D6's qualifier in Scope.

### [LOW] No criterion pins that head answers stay out of canonicalization

**Location:** `_lyx/discussion/decision-record.md:137`.
**Issue:** D5 requires each head's answer to be discarded.
If the plan builds the head sources into the same `canonicalOwners` map, two cards that re-sign the same member produce two owners of one canonical form and a false `handle-canonical-collision`.
The acceptance criterion covers one arrow bullet that validates and the three refusal shapes, not this case, and the chokepoint pin cannot detect it.
**Fix:** Add to the arrow criterion that the same member re-signed by two cards validates clean and the plan on disk is not rewritten because of an arrow.

### [LOW] The Rename to-side criterion has one direction that matters

**Location:** `_lyx/discussion/decision-record.md:135`.
**Issue:** Both endpoints of a `Rename` pair project into the card's targets, so the Old side, a member glyph in its own unit, already trips the pairing whenever the package self glyph names that unit.
That direction gives the same verdict under the Old-glyph-unit rule and the draft-spelling rule.
The two rules differ only when the draft handle spells the package self glyph's unit and the Old glyph's unit is another: the Old-glyph-unit rule gives no finding, the draft rule a false one.
The criterion's "exactly when" implies it, but a plan writer can satisfy the sentence with the first direction alone.
**Fix:** Name the second case in the criterion: a self glyph of the package the draft handle spells, with an Old glyph in another unit, gets no finding at the plan gate or from `ValidateDispatch`.

### [NIT] D5 takes a quarry unit from `Glyph.UnitPath`

**Location:** `_lyx/discussion/decision-record.md:74`.
**Issue:** `quarry.Declaration.Unit` is a glyph unit; `renameDeclSource` already passes `sym.Glyph.Unit`, and `HandleUnit` passes the handle's unit half.
`Glyph.UnitPath` is the glyph-to-path call and returns ok false for a non-path alphabet.
For Go the two strings are equal, so nothing breaks, but the wording sends the plan writer to the path conversion for a non-path purpose.
**Fix:** Say "under the unit of its glyph" and drop `Glyph.UnitPath`.
