---
round: 1
focus:
  - "Issue 1, `internal/gatecli/tierrun.go`: does the no-test line cover the skipped-class report the board entry asks for (which classes the gate skipped under COMMONLIB_REPO and why), or only a vstest run that reported no test? Check the diff against each bullet of the entry, and the false-positive and false-negative cases of the zero-test detection."
  - "Issue 1, stencils `darn-fragment-make-change.md` and `webster-body-implementer.md`: does the new wording tell agents how to run gate-skipped tests by hand, and does it contradict the stencils' existing rule on raw test commands?"
  - "Issue 2, `internal/fabricengine/weftwiring.go` `CodeLocation` and `resolveHub` in gatecli: is the records-to-code mapping correct for the prime, a task worktree and a non-records path, and does it fail safely when no pair exists?"
  - "Issue 2 second bullet: does the diff check the other refusals that take their way forward from `hubgeom.PrimeWorktreePath` for the weft-for-code mix-up, or is that check only claimed in the commit message?"
  - "Issue 3, `detachfront_integration_test.go` and `redundancy_integration_test.go`: does each change remove the named race, or only widen a timing margin, and does the new lease or timeout handling leave a path that hangs or leaks a slot?"
  - "Test economy: `TestRunTierNoTestNotice` and `TestRunInvocationsTestTargets` are listed as unique; check that the new tests in `tierrun_test.go` and the `testmap_integration_test.go` addition cover each new behavior and none duplicates an existing test."
  - "Doc comments and `internal/gatecli/doc.go`: do the changed comments follow the repository's constraints document, and does any doc the repository names for this kind of change need to land in the same diff?"
---

Seed focus drawn from the board entry's three issues and the commit range's file list.
