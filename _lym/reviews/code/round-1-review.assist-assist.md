---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/editdirective/editdirective_test.go:41
    summary: The scratchBuildBan assertion pins static stencil prose contrary to PATTERN-test-speed.
    fix: Remove the scratchBuildBan constant and its strings.Contains assertion from TestDirective_ReadsTheEnginesCopyWithoutBanner in internal/editdirective/editdirective_test.go, and remove the scratch-ban claim from its introductory comment. Keep the existing engine-selection and rendering assertions.
    origin: lens:code-tests
---

### [BLOCKING] Remove the static-prose assertion

**Location:** internal/editdirective/editdirective_test.go:41

**Issue:** The new assertion requires the literal sentence declared at lines 20–21 in both rendered engine copies.
`pattern/PATTERN-test-speed.md:250` prohibits tests pinning static stencil prose and reserves wording checks for review.
This sentence is neither a marker contract nor a value selected or filled by Go.
Repeating the check for both engines does not make it a structural rule across a stencil family.
The code-tests and code-affected-set lenses reported the same defect; this finding consolidates them.
The exploration instructions require a listed constraint violation to be BLOCKING.

The other focus checks found no defect.
Both engine directives cover build, restore, compile, worktree, clone and checkout under all three named folders.
The removed build-resolution workaround is superseded by the requested ban.
The orch role carries the same prohibition, and its start, resume and reload pointers require reading the role in full.
The retro command's work directory holds report intermediates; its script reads existing repositories without creating checkouts.
The shipped-skill search found no contrary build instruction.
The package documentation and landing summary describe the diff, and the fabric documentation remains unchanged.
The `.scratch` entry remains a link into the task's weft.

Validation used source reads and read-only command checks.
No build or test suite was rerun; the supplied review input records the gate's redundancy report against HEAD.
