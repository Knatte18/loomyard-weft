---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-correctness
    location: contracts/stencils/webster/webster-body-implementer.md:58
    summary: The sweep sentence calls an out-of-target edit a fix to a failure the card caused, the second sanctioned deviation, but the sweep runs before any failure, so the two-pronged test and attempt cap at lines 43 and 119-124 cannot bound it.
    fix: Reword line 58 so an edit to a hit outside the card's targets is sanctioned only when the hit references a name the card retires, stating that the edit stays limited to those references and is not a failure fix under the self-fix cap. Keep the first/second ordering of the sanctioned deviations at lines 43 and 69 unchanged, or extend line 43 to name the sweep edit as its own bounded case.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/websterengine/template_test.go:810
    summary: The two added contains assertions pin static stencil prose that no template action can change, and the sibling assertion on the quarry-resolve line already proves the fork stencil renders.
    fix: Delete the two added contains lines at template_test.go:810-811 and restore the case title at line 801 to "fork reads the card files, locates by quarry and ends on the batch report".
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-body-implementer.md:55
    summary: The phrase after any rename tool run repeats the grep the Rename paragraph at line 49 already requires after a rename tool run, and does not say whether one grep serves both.
    fix: In line 55, say that for a symbol rename the grep at line 49 is the sweep's single whole-worktree grep, widened to test files, testdata and fixtures, so the fork runs one grep rather than two.
---

### [MEDIUM] Sweep edit is called a failure fix, so no bound applies to it

**Location:** `contracts/stencils/webster/webster-body-implementer.md:58`
**Issue:** Line 43 defines the second sanctioned deviation as a fix to a failure the card caused, under the self-fix section's two-pronged test and its attempt cap.
Lines 119-124 require an observed failing test or a scan flagging a file the card wrote.
The sweep runs before the card's first hand edit, so no failure exists, and the sentence labels every out-of-target hit as that deviation.
A hit that matches a retired string by accident, such as a generic literal, is licensed the same way.
The numbering of the first and second deviations is consistent (lines 43 and 69).
The unbounded licence is the defect, and the wording matches the stated aim of the board entry only in part, since the entry asks for editing every hit and says nothing of out-of-target scope.

### [LOW] Added assertions check static stencil prose

**Location:** `internal/websterengine/template_test.go:810`
**Issue:** The strings at lines 810-811 match stencil text outside any template action.
The existing assertion at line 809 already covers that the fork stencil renders, so the added strings add no behavioural coverage.
The test-economy rule names a new assertion on a stencil's static prose as a finding.
The renamed case title at line 801 describes the added assertions accurately, and is reverted only because the assertions go.

### [LOW] Sweep grep overlaps the Rename paragraph's grep

**Location:** `contracts/stencils/webster/webster-body-implementer.md:55`
**Issue:** Line 49 runs the rename tool, then greps for zero references to the old name.
Line 55 sweeps "after any rename tool run", so a symbol rename gets two greps with no word on whether one satisfies the other.
The order does not conflict, so the fork can follow both, at the cost of the extra turn the entry aims to save.
The grep at line 56 and the "instead of grepping" at line 53 are not in conflict: the first is scoped to retired names, the second to locating the card's own glyphs.
A file pair renamed by `git mv` is also not a rename tool run, and "after any rename tool run" is silent on it.

## Rejected

- Scope: the diff is one stencil paragraph and one test case, within the board entry's stencil-only change, and no other stencil or code restates step 2, so no further sweep text is needed.
- Missing elements: all four elements of the board entry are present, namely the whole-tree grep including tests and fixtures, before the first edit, one read message, one edit message.
- Summary claim: the change description's design notes agree with the stencil text, so it holds, apart from the bound on the second deviation raised as F1.
- Commit message and PATTERN: the commit carries no `Pattern-Departure:` trailer and none is needed, and the focus directive declares no PATTERN change to land.
