---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: contracts/specs/loom-plan-checks-spec.md:161
    origin: handler
    summary: "seam-test-unlisted entry drops the package-counterpart coverage rule that internal/loomshed/seamtest.go applies, so the spec states a narrower covering condition than the gate"
    fix: "In the seam-test-unlisted entry of contracts/specs/loom-plan-checks-spec.md (line 163, the 'It covers the test when' sentence), add that a card also covers the test when it names the test glyph's package counterpart (the same member in the other Go package of the directory, <dir> versus <dir>_test), and when a ref is the package self glyph of that counterpart unit. Keep it one clause."
  - id: F2
    severity: LOW
    class: design
    location: contracts/specs/loom-plan-checks-spec.md:103
    origin: handler
    summary: "The split dropped the Bound notes that told a plan reviewer what a passing check still leaves to review, for caller-uncovered, seam-test-unlisted, doc-go-unlisted and test-tier-unstated"
    fix: "In contracts/specs/loom-plan-checks-spec.md add one short sentence per entry, taken from the base version of loom-plan-spec.md (git show 51bd6281, rows caller-uncovered, test-tier-unstated, seam-test-unlisted, doc-go-unlisted). caller-uncovered misses a caller in a file an earlier card creates and an interface break at an assignment site. test-tier-unstated does not judge that the stated tier fits. seam-test-unlisted passes a test listed under Uses while the card breaks it. doc-go-unlisted passes a doc.go under Extends or Uses that the card falsifies, and counts a card that creates a test helper by a handle without a Test, Benchmark, Fuzz or Example prefix as non-test code. State each as what the gate does not see, in one clause."
  - id: F3
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-rubric-plan-review.md:49
    origin: lens:code-affected-set
    summary: The rubric says ImpactSummary is required for Edit and Delete only, citing the plan spec table, which now requires it for Extends too
    fix: "In contracts/stencils/loom/loom-rubric-plan-review.md line 49 change 'It is required for `Edit` and `Delete` only' to 'It is required for `Edit`, `Extends` and `Delete` only', matching the table at contracts/specs/loom-plan-spec.md:116 and checkCardMissingField."
  - id: F4
    severity: LOW
    class: design
    location: contracts/stencils/webster/webster-template-plan-fix.md:23
    origin: lens:code-affected-set
    summary: The cold plan fixer is pointed at the plan spec and the checks spec but not at the C# spec, which alone holds the way forward of several C# checks
    fix: "In contracts/stencils/webster/webster-template-plan-fix.md step 3, after the checks-spec sentence, add a sentence that on a plan whose overview sets language csharp the C# rules are in {{.specs_dir}}/loom/loom-plan-csharp-spec.md. The stencil already declares the specs_dir marker, so no new marker is needed."
---

### [LOW] seam-test-unlisted states a narrower covering condition than the gate applies

**Location:** contracts/specs/loom-plan-checks-spec.md:163
**Issue:** The base spec said a card covers a marked test when its target groups name the test's glyph "or the glyph's package counterpart", or a ref is the package self glyph "of the test's unit or of its counterpart unit". The new entry keeps only the test's own glyph and unit. `internal/loomshed/seamtest.go:137` still calls `planparser.GoPackageCounterpart`, so a writer who lists the external-test spelling of the test is accepted by the gate and refused by the spec's wording. The move cut a rule a writer acts on.

### [LOW] Bound notes that mark what a passing check leaves to review were cut

**Location:** contracts/specs/loom-plan-checks-spec.md:103
**Issue:** The Plan-Review rubric tells the reviewer not to re-derive what the gate checks, so the reviewer relies on the spec for where the gate is blind. The base entries for `caller-uncovered`, `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted` each closed with a Bound naming such a case; the new entries keep none. The decision record asks to cut rationale a reader does not act on, but these are the cases a reviewer still has to judge by hand, and the summary's "long Bound paragraphs" cut took them whole instead of shortening them.

### [LOW] The ImpactSummary rule in the plan-review rubric contradicts the surviving table

**Location:** contracts/stencils/loom/loom-rubric-plan-review.md:49
**Issue:** The line cites the per-type table of `loom-plan-spec.md` for "Edit and Delete only". The table (line 116) and the field list (line 84) require it for `Extends` too. The base spec held the same contradiction between its prose and its table; the diff cut the prose, so the rubric now cites a source that disagrees with it.

### [LOW] The plan fixer is not pointed at the C# spec

**Location:** contracts/stencils/webster/webster-template-plan-fix.md:23
**Issue:** Before the split the card format and the C# rules sat in one file the fixer read by part. Now the fixer is told to read the plan spec and the checks spec. A fixer on a `language: csharp` plan fixing `resign-head-mismatch`, `rename-namespace-pair` or `resign-overload-collision` reads those entries' pointer to the C# spec only for some of them, and the way forward of the rest lives only in `loom-plan-csharp-spec.md`.

## Rejected

- Lens code-correctness reported no finding, and the handler agrees: the embed variables, registry rows and registry tests are consistent, and the other edits are comments and one raw-string help text.
- Lens code-tests reported no finding, and the handler agrees: `TestRegistry_NamesAndDefaults` and `TestRegistry_RelPathPlacesBothUnderLoomFamily` pin the two new registrations, and the `validate_test.go` edit changes comments only, so its assertions are as before.
- Handler candidate, dropped: the `Move` false-positive `path-missing` rule appears in both `loom-plan-spec.md` and `loom-plan-checks-spec.md`. One copy is the rule a writer follows and the other the check's entry, so neither is dead text.
- Handler candidate, dropped: every check ID the base spec's validation, part-scope and joined-plan sections named is still present in the four new files (compared mechanically, only the verb `begin-batch` was not carried, and it is not a check).
- Handler candidate, dropped: the worked example's changes (card 1 as a `plan:` handle, a `lym gate test --wait` Verify line, no `## verify:` section) match the summary's claim and obey the rules the spec states.
- Handler candidate, dropped: the summary's size figures (24 KB, 19 KB, 5 KB) match the files; "Together 57 KB" matches the three plan files plus the larger `loom-parts-spec.md` only if read as four files, which the sentence does not say, so it is imprecise rather than false.
- Handler candidate, dropped: the `planglyph/doc.go` and `validate.go` comment re-breaks are line-break changes, which the rubric bars as findings.
