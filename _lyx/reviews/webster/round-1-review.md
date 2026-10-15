---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: contracts/stencils/loom/loom-rubric-plan-review.md:62
    summary: Dropping the test-file sentence from the Member glyphs bullet joined the next bullet onto its last line, so the re-sign arrow bullet is gone from the rubric's list
    fix: In contracts/stencils/loom/loom-rubric-plan-review.md, split the line that ends "the finding names the member glyphs to list instead.- **The re-sign arrow.**" so that "- **The re-sign arrow.**" starts its own line at the top-level bullet indent, directly after the "**Member glyphs on an `Edit` card.**" bullet's one remaining sentence line. Then run `lyx gate test ./contracts/stencils` to confirm the rubric claims still pass.
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-template-plan.md:148
    summary: The Edit section now says twice that a new test file is a Create path and twice that a new test function is a Create plan handle
    fix: In contracts/stencils/loom/loom-template-plan.md, delete the older two sentences "A new test function in an existing file is a `**Create:**` declaration with a `plan:` handle." and the later "A new test file is a `**Create:**` path.", keeping "A new test file is a `**Create:**` path, written only for a surface no existing test file covers." and the new "A new test function is a `**Create:**` declaration with a `plan:` handle whose unit is the package directory, ..." sentence. Then run `lyx gate test ./contracts/stencils` and keep the wantAll claim strings in contracts/stencils/claims_test.go matching.
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD`, since `_lyx/loom/rework/` does not exist and `product.parent` in the run status is `main`.
`lyx gate test` over `internal/planparser`, `internal/planglyph`, `internal/loomcli`, `contracts/stencils` and `cmd/lyx` passes.

### [BLOCKING] Rubric edit merged the re-sign arrow bullet into the previous line

**Location:** contracts/stencils/loom/loom-rubric-plan-review.md:62
**Issue:** Card 3 asked only for the sentence about a test-file-only `Edit` card to be dropped.
The edit also removed the newline before `- **The re-sign arrow.**`, so line 62 now reads "...to list instead.- **The re-sign arrow.**".
In the deployed Plan-Review rubric the re-sign arrow rule is no longer a bullet of its own: it renders as stray text inside the Member glyphs bullet, and its body lines now hang under that bullet.
The rubric is a prompt read by a reviewer, so the structure matters, and the existing wording claims do not pin it, which is why the tests stay green.

### [LOW] Duplicate test-file rules in the plan stencil

**Location:** contracts/stencils/loom/loom-template-plan.md:148
**Issue:** The rewrite added "A new test file is a `**Create:**` path, written only for a surface no existing test file covers." and "A new test function is a `**Create:**` declaration with a `plan:` handle whose unit is the package directory...".
The older "A new test function in an existing file is a `**Create:**` declaration with a `plan:` handle." and "A new test file is a `**Create:**` path." were left in the same section.
Each rule is stated twice a few lines apart, and the prose rule is to say it once.

## Checks with no finding

- `edit-test-file` is appended at both `ValidateFormat` (told the done set) and `ValidateRework` (nil), and not at `ValidateDispatch`; this matches card 1 and the spec row 43.
- No branch of `coverageVerbAndWay` or the `redundant-file-target` detail can still offer a `_test.go` file under `Edit`; the stencil no longer shows a plain test-file path under `Edit`.
- The `Glyph.UnitPath` sole-caller wording in `internal/planparser/doc.go` and the spec matches the code.
- Test economy: `TestCheckEditTestFile` is the one new top-level test and the `internal/planparser` budget rose by exactly one; the other additions are rows or assertions in existing tests.
