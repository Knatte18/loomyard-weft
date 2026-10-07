---
round: 2
focus:
  - "Comments added or edited in the review range, including those in internal/planindex/planindex.go, internal/planglyph/repo.go, internal/websterengine/beginbatch.go and internal/websterengine/runlevel.go: settle whether each breaks only at sentence ends, semicolons or a comma before a coordinating conjunction, by running `lyx loom lint-comments` built from this branch against the merge base with main."
  - "The scenario and guard tests in internal/commentlint, internal/impactset, internal/loomcli and cmd/lyx integration test files: settle whether each calls t.Parallel or carries a comment naming the process-global state that prevents it."
  - "internal/commentlint/lint.go `isProseBlock` and the `//lyx:guard` and `//testtiming:keep` markers stacked into doc comment blocks: settle whether wrapped prose above a directive line is still flagged, and whether doc.go and the lint's tests state that behavior."
  - "The coordinating-conjunction set in internal/commentlint/lint.go: settle whether a line ending in a comma before a conjunction that starts an independent clause is accepted, while introductory phrases after a comma are still flagged."
  - "internal/loomshed/verifygate.go and the errors returned by impactset.Derive: settle whether a git or directory read failure is returned as an error rather than handed to the writer as a finding, and whether each finding the writer does get names the file, the line and the move that fixes it."
  - "contracts/specs/refusal-spec.md rows for the Webster-Burler verify gate: settle whether every refusal the gate can now make (failing tests, comment lint findings, misplaced guard markers) has a condition and a way forward in the spec, and a test reaching it."
  - "cmd/lyx/testdata/test-budget.yaml against the top-level test count of each package under every build tag: settle whether every row equals its package's count."
  - "internal/websterengine Geometry with no code index: settle whether the begin-batch path that reaches a nil Index returns an error naming the missing wiring, and whether a test reaches it."
  - "cmd/lyx/treesitterlink_test.go and treesitterlink_integration_test.go: settle whether any test pins an unexported helper that another test in the package already covers over real `go list` output."
---

Each entry restates the site and the question; none depends on an earlier round's review.
