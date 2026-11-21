---
round: 1
focus:
  - "internal/gatecli/redfirst.go `baseOverlay`, `embedPatterns`, `embedPatternMatches`, `embeddedAtBase`: settle whether a changed non-Go file that a base production Go file embeds is always restored to its base text, and every other non-Go file keeps its working-tree text. Check embed pattern forms (directories, `all:` prefix, globs, multiple patterns per directive, files absent at base, files added at HEAD) and the failure path when a base Go file cannot be parsed."
  - "internal/gatecli/redfirst.go `redFirstCall.run` and internal/loomshed/redfirstgate.go `redFirstFailure`: the reach report shows no mapped test reaches them. Settle whether the verdict, record and exit code for the unjudged, red and green outcomes still follow from the changed code, and whether the unjudged text now comes from the scoper alone as the last commit states."
  - "Commit 4abc6f41 `Root-Cause:` trailer: settle whether the defect it names is the one the diff removes, and whether the trailer and the commit message claims hold against the diff."
  - "Waiver in the decision record: the stencil edit `contracts/stencils/darn/darn-template-write.md` is licensed by the parent entry. Settle whether the stencil change drops, weakens or contradicts any rule the Goal did not ask to change."
  - "Docs that must land with the change (`docs/overview.md`, `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `internal/gateslot/redfirst.go`): settle whether they describe the new overlay behavior and no longer claim non-Go files waive the gate."
  - "Tests `internal/gatecli/redfirst_test.go`, `redfirst_integration_test.go`, `internal/loomshed/redfirstgate_test.go`: settle whether any assertion at the review base was widened or removed, whether each test still checks what its name states, and whether expected values are computed independently of the code under test."
---

Seed focus for round 1, set from the decision record, the range's commit messages and the reach report.
