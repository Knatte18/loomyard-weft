---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/planglyph/doc.go:39
    summary: Changed Go comments keep fixed-column wraps that break sentences mid-phrase, against PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/planparser/redundancy.go:124
    summary: redundant-package-target keys a member by Glyph.Unit and a package or file self glyph by its UnitPath and compares the two, reading Unit as a disk path against PATTERN-glyph-conversion-chokepoint
  - id: F3
    severity: MEDIUM
    class: consistency
    location: contracts/specs/loom-plan-spec.md:248
    summary: Rewritten spec paragraph holds several sentences on one line, against the repository's semantic-line-break rule for .md files
  - id: F4
    severity: LOW
    class: design
    location: internal/planglyph/plangate.go:62
    summary: A target listed twice on one card yields duplicate redundant-file-target and caller-uncovered findings, where the plan and the spec say one finding per file and member or per subject and file
  - id: F5
    severity: LOW
    class: design
    location: internal/planglyph/plangate.go:104
    summary: isInterfaceMethod and renameDeclSource detect an interface method by a signature that does not start with func, which an interface method named funcName defeats
  - id: F6
    severity: LOW
    class: scope
    location: internal/planglyph/callercoverage.go:149
    summary: The directory skips (testdata, vendor, dot- and underscore-prefixed) in scanReferences have no test
  - id: F7
    severity: LOW
    class: consistency
    location: internal/planglyph/doc.go:12
    summary: Package overview still says Validate and ValidateFormat append only resolvePass findings and omits planGatePass
  - id: F8
    severity: LOW
    class: consistency
    location: internal/planparser/validate_test.go:1148
    summary: The row a path beside a package self glyph is clean pins an input ParsePlan never produces, since a path with an extension is canonicalized to a file self glyph, which the previous row flags
  - id: F9
    severity: NIT
    class: consistency
    location: internal/planglyph/glyphchain_integration_test.go:163
    summary: The plan-gate assertion repeats the CrossCard row create handle no other card references
---

Range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which is cards 1-12.
Evidence run: `go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...`, `go test -tags integration ./internal/planglyph/`, `lyx loom lint-comments --base fd5b0f58e` (clean) and `gofmt -l` over the touched trees (nothing touched).
Probes ran in a scratch copy of the tree outside the repository.

### [BLOCKING] Changed Go comments keep fixed-column wraps

**Location:** `internal/planglyph/doc.go:39-42`; also `internal/websterengine/beginbatch.go:275`, `internal/planglyph/repo.go:55`, `internal/planglyph/planglyph.go:83`, `internal/planglyph/handle.go:89`, `internal/planglyph/handle.go:364`
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line and allows a break inside a sentence only after a semicolon or at a comma before a coordinating conjunction.
It also says a fix rewraps the sentence the edit touches, and `lint-comments` enforces only part of the rule.
`doc.go:39-42` is a sentence the diff wrote from scratch, and it breaks after `create.go and`, after `at its own` and after `across two of them is`.
The other locations are sentences the diff edited and whose old column-wrapped tail it left: `beginbatch.go:275` ends `passes both run after the`, `repo.go:55` ends `or looks`, `planglyph.go:83` leaves the orphan line `// The pure`, `handle.go:89` ends `when Old did not`, and `handle.go:364` ends `plan:-prefixed`.
**Fix:** Rewrite each touched sentence on one line, or break it only at a clause boundary; leave untouched neighbouring lines alone.

### [BLOCKING] redundant-package-target reads Glyph.Unit as a disk path

**Location:** `internal/planparser/redundancy.go:124` (and `:93`, `:126-135`)
**Issue:** `PATTERN-glyph-conversion-chokepoint` forbids reading `Glyph.Unit` as a disk path.
`redundancyOwnerOf` keys a member glyph by `g.Unit`, but keys a package self glyph by `unitPath` from `UnitPath()` and a file self glyph by `path.Dir(unitPath)`.
`checkRedundantPackageTarget` then matches the three in one `containers[m.unit]` lookup, so it is correct only because a Go unit happens to equal the path.
`renameToSideUnits` stores `old.Unit` in the same map.
The code this card set replaces compared `Unit` with `Unit` and kept `unitPath` apart.
**Fix:** Take the member's key from `g.UnitPath()` as well, and `old.UnitPath()` in `renameToSideUnits`, so every key in the map is a `UnitPath` result.

### [MEDIUM] Rewritten spec paragraph holds several sentences on one line

**Location:** `contracts/specs/loom-plan-spec.md:248`; also the edited row 11 `glyph-malformed` line
**Issue:** The repository rule is one sentence per line in every `.md` file.
The paragraph beginning `The handle checks do not all read the same source set` was rewritten in this diff, and it adds the sentence `A declared handle no other card references is valid: ...` to a line that already holds three more.
Row 11 was edited in place and still carries several sentences on one line.
**Fix:** Put each sentence of the rewritten paragraph on its own line, and do the same for the row 11 sentences the edit touched.

### [LOW] Duplicate findings for a target listed twice

**Location:** `internal/planglyph/plangate.go:62-63`, `internal/planglyph/callercoverage.go:107`
**Issue:** Card 8 says one finding per file and member, and card 11 says one finding per subject and file.
Neither `planGatePass` nor `coverageSubjects` deduplicates, while `checkRedundantPackageTarget` does with its `seen` map and tests that.
A probe with `shapes/shapes.go`, `shapes#Func`, `shapes#Func` on one Edit card gave two identical `redundant-file-target` findings, and `callees#Target` twice under one Delete group gave six `caller-uncovered` findings for three files.
**Fix:** Skip a repeated ref when building `files` and `members` in `planGatePass`, and a repeated `(card, ref)` in `coverageSubjects`, and add a row to the existing test of each check.

### [LOW] Interface-method detection by the `func` prefix misfires on a method named func...

**Location:** `internal/planglyph/plangate.go:104`, `internal/planglyph/handle.go:142`
**Issue:** Both detect an interface method as a method whose signature does not start with `func`.
quarry answers an interface method named `funcName` with the signature `funcName() int` (checked with a probe against `quarry.Resolve`), which starts with `func`.
So `resign-interface-method` does not fire on it, and `renameDeclSource` skips the `func (Owner) ` prefix and hands `quarry.Name` a bare spec.
I read the second consequence from the code and did not run a rename.
**Fix:** Test for `func (`, which every non-interface method signature carries, in both places.

### [LOW] The scan's directory skips are untested

**Location:** `internal/planglyph/callercoverage.go:149`
**Issue:** `scanReferences` skips `testdata`, `vendor` and directories starting with `.` or `_`, and card 11 states the rule, but no row puts a caller in such a directory.
Comments, string literals, a selector on another package's struct, the external test package and the same-file reference are covered; the skips are the one stated behavior with no test.
`PATTERN-test-economy` counts a behavior the diff adds with no coverage as a gap.
**Fix:** Add one case to `TestGlyphChain_CallerCoverage` (a fixture file under a skipped directory name, written into the fixture copy by the test) that expects no finding for it.

### [LOW] Package overview omits planGatePass

**Location:** `internal/planglyph/doc.go:12`
**Issue:** The overview says `ValidateFormat` and `Validate` append "the same resolve-backed findings on top via one shared resolvePass".
They now also append `planGatePass`, which `ValidateDispatch` does not; only the per-check bullets mention it.
**Fix:** Name `planGatePass` in that paragraph, with the plan-gates-only split.

### [LOW] A test row pins an input the parser never produces

**Location:** `internal/planparser/validate_test.go:1148`
**Issue:** The row `a path beside a package self glyph is clean` builds a card by hand with the raw target `pkg/a/file.go`.
`ParsePlan` canonicalizes a path with an extension to `pkg/a/file.go#` (the golden fixture expectations in `parse_test.go` show it), and the row above this one flags that spelling.
The row therefore asserts a clean verdict for a shape real plans cannot reach.
**Fix:** Drop the row.

### [NIT] Repeated plan-gate assertion

**Location:** `internal/planglyph/glyphchain_integration_test.go:163`
**Issue:** `TestGlyphChain_UnreferencedHandle` opens with `assertPlanGate(t, plan, f.root, nil)` on the same plan that `TestGlyphChain_CrossCard` already pins clean in the row `create handle no other card references`.
The delta, bind and done-check part is not covered elsewhere and stays.
**Fix:** Drop the repeated assertion.

## Focus verdicts

These settle each seed focus entry; entries with no finding above are not defects.

- Caller coverage walk: package-level versus method rules, comments and string literals, the `<package clause>.N` form for an external test package, and exclusion of the subject's own spans match card 11; the fixture rows pass.
  The directory skips match the card and are untested (F6).
  A repeated target duplicates findings (F4).
- Plan-gate-only placement: `planGatePass` has exactly two callers, `ValidateFormat` and `Validate`; `ValidateRework` reaches it through `ValidateFormat`.
  `redundant-package-target` and `resign-not-member` run in `planparser.validate`, and `resign-head-mismatch` runs in `CanonicalizeHandles`, so all three run at every gate.
  The dispatch assertions in `TestGlyphChain_*` pin the split. Not a defect.
- Re-sign head matching: a re-sign source is compared and `continue`d before `canonicalOwners`, so it never reaches the collision check or `RewriteRefs`; a two-card re-sign leaves both card files unchanged, and the pointer-receiver and changed-receiver rows both pass. Not a defect.
- Retirements: no reference to `containment-unit-overlap`, `containment-file-overlap`, `handle-unreferenced`, `resolveContainment`, `syntacticContainment`, `writingTargetCards` or `declaredHandles` remains in code, `doc.go`, the spec, the stencils or the tests apart from the intended `claims_test.go` negative pin.
- Spec renumbering: the three `row N` cross-references (rows 3, 13 and 26) still point at the right rows, the new rows follow the order the code runs them in, and the Edit and Delete mechanical-check cells name `caller-uncovered`. Not a defect, apart from the line breaks in F3.
- Per-card mechanical checks: only card 5 carries a Delete group (four files); the build and all four `go vet` tag sets are clean, so assert-no-callers holds for each of its targets. No card carries a Rename group.
- Test economy: each added check has a blocking and a clean outcome; the only redundancy found is F8 and F9, and the only gap is F6.
- Comment conventions: see F1; `lint-comments` passes but does not cover the cases listed there.
- Stencil and rubric: `claims_test.go` pins the new text and the removed `handle-unreferenced` text, and the stencil points at the spec's check IDs.
