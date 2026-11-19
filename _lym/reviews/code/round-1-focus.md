---
round: 1
focus:
  - "internal/gatecli/redfirst.go, internal/loomshed/redfirstgate.go: does the gate fail only when every changed test passes on the merge-base tree, and does a build failure or a failure outside any test (`PackagesFailedOutsideTests`) count as red or as an error, as the board entry's intent requires?"
  - "internal/gatecli/redfirst.go `baseOverlay`, `writeOverlay`, `changedTestsAt`: does the overlay replace only the production Go files of the module and keep the diff's test files, so the merge-base run really tests old code with new tests; are added, deleted and renamed files handled?"
  - "internal/loomshed/redfirstgate.go and internal/shedrecipe/entries_gate.go: is the gate applied only to an entry labelled `bug`, and is a bug with no test seam a named stop rather than a silent pass?"
  - "contracts/stencils/darn/darn-template-write.md, contracts/stencils/loom/loom-rubric-code-review.md, contracts/recipes/darn-recipe.yaml: does the `Root-Cause:` trailer reach Code-Review as a check like `Loosened-Check:`, and do the stencil edits drop, weaken or contradict any rule the board entry did not ask to change?"
  - "Test economy and the mutation survivors at internal/gatecli/redfirst.go:287 (`changedTestsAt` condition) and :353 (`writeOverlay` constant), and internal/gateslot/runlog.go:424: is there a check the tests lack for the diff's own behavior?"
  - "Docs and claims: do docs/overview.md and the package doc.go edits match what the code does, and does the change description (when present) claim anything the diff does not bear out?"
---

Seed focus drawn from the board entry in the decision record and the diff stat of the review range.
