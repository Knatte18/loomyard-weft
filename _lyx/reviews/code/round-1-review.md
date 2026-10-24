---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/websterengine/fingerprint.go:60
    summary: webster's plan fingerprint and per-file hashes now fold in the lens reports saved beside lens-review.md
    fix: In planFileNames, also skip a file for which planparser.IsLensReportFileName(e.Name()) holds, update the doc comments of fingerprint and planFileNames to say the lens reports are records about the plan like the lens review, and add a fingerprint test row with a lens-review.lens-<lens>.md present that shows the fingerprint and planFileHashes unchanged.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/hubgeom/hubgeom.go:141
    summary: The SpecsDir fill in hubgeom's and standalonegeom's Burler geometry has no test, so dropping it passes the suite and fanned prompts name a relative spec path
    fix: Extend TestBurlerGeometry in internal/hubgeom/hubgeom_test.go to assert SpecsDir equals fabricengine.SpecsDir of the hub path, and TestBurlerGeometry in internal/standalonegeom/standalonegeom_test.go to assert SpecsDir equals SpecsDir(stateDir); add no new test functions.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/review.go:99
    summary: Engine.Review forwards e.geom.SpecsDir but no test drives a fanned Engine.Review and checks the spec path, while Engine.Fix is pinned through the engine
    fix: In an existing fanned Engine.Review test in internal/burlerengine, set SpecsDir in the Geometry and assert the review step names stencilstore.Path(specsDir, lensreport.SpecName).
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/stencils/burler/burler-template-reviewer.md:23
    summary: The reviewer template is shared by solo rounds, so its write-surface and closing lines now mention lens reports in a solo round's prompt, which the acceptance criteria say stays unchanged
    fix: Move the lens-report allowance out of the shared template into the cluster-fan part, which already says the lens reports are part of the handler's write surface, and restore the template's two lines "Your write surface is the review file `{{.review_path}}` and nothing else." and "You write no other file and send no message."; if the fan part's sentence is not enough to lift the template's "nothing else", add a marker the engine fills empty for a solo round.
---

### [MEDIUM] Webster fingerprint includes lens reports

**Location:** internal/websterengine/fingerprint.go:60
**Issue:** `planFileNames` hashes every `.md` in the plan directory except `amendments.md` and `lens-review.md`, whose doc comment excludes the lens review because it is a record about the plan. The reports `lens-review.lens-<lens>.md` are the same kind of record and belong to the same generation (D7), but the diff teaches `planparser` and the rotator about them and leaves this third consumer of the plan directory untouched. They now enter plan identity and the per-file hashes.

### [MEDIUM] SpecsDir fill in the two Burler geometry builders is untested

**Location:** internal/hubgeom/hubgeom.go:141, internal/standalonegeom/burlergeom.go:29
**Issue:** D5 has both builders fill `SpecsDir`. The only `SpecsDir` assertions in those packages are for the Webster geometry. A dropped line leaves the field empty and fanned prompts name a relative spec path with no test failing.

### [LOW] Engine.Review SpecsDir forwarding untested

**Location:** internal/burlerengine/review.go:99
**Issue:** The prompt tests call `composeReviewPrompt` with a hand-passed directory, so a wrong field in `Engine.Review` passes. The fix side is pinned through `Engine.Fix`.

### [LOW] Solo reviewer prompt text changed

**Location:** contracts/stencils/burler/burler-template-reviewer.md:23
**Issue:** The acceptance criteria keep a solo round's prompt unchanged. The reviewer template is rendered for every round, so a solo reviewer now reads "plus the lens reports a cluster round's rules name" and "beyond those lens reports".

## Rejected

- code-affected-set F2 (non-overlay fixer write surface may forbid the reports): rejected, the source-scope write surface is the whole task worktree, and D6 names only the overlay-scope fixer.
- code-tests NIT (host tests match stencil sentence fragments): rejected, the assertions pin the value each host passes and the fork itself would keep them.
- code-correctness and code-pattern-fit returned no findings; the naming package, the `index-file-mismatch` skip, the rotation announce filter, the spec format and registration, the PATTERN-leaf-packages entry and the docs all hold against the fasit.
