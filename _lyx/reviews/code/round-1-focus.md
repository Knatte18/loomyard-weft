---
round: 1
focus:
  - "internal/lensreport/lensreport.go and its tests: does the path pattern and the report-name predicate match D1's four record-to-report names, and recognize exactly those names (no prefix or suffix over-match, e.g. a record file or a Burler round file)?"
  - "internal/planparser/validate.go and amendment.go: does index-file-mismatch skip only lens-review.lens-*.md, and does Plan-Write's rotation in internal/loomshed/planwrite.go move a report without naming it in the prior-plan block (D7)?"
  - "internal/burlerengine prompt.go, fix.go, review.go and geometry.go with hubgeom and standalonegeom: is SpecsDir told through geometry and never derived, and are a solo round's and a fan-less fixer's prompts unchanged (acceptance criteria)?"
  - "Burler stencils (burler-cluster-fan, burler-fix-self-check, burler-template-reviewer) and twill stencils: do they point at the deployed spec path without restating its format, and do the reviewer and overlay-scope fixer write surfaces name the lens reports (D3, D4, D6)?"
  - "contracts/specs/twill-lens-report-spec.md and specs.go: is the format a fenced YAML header over the verbatim final message, with a no-report outcome and no body, and is it registered under the twill family?"
  - "Docs and PATTERN: do doc.go files, docs/overview.md and pattern/PATTERN-leaf-packages.md land the new package and behavior, and do touched markdown files and comments follow semantic line breaks?"
  - "Test economy: do the new tests in lensreport, planparser, loomshed, loomengine, twilldirective and burlerengine each add coverage not already given by a public-surface test, and is every added behavior covered?"
---

Seed focus from the decision record's scope and acceptance criteria; the diff is two commits over the range base.
