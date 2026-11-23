---
round: 1
focus:
  - "internal/orchcli/notify.go PostAsCaller, postSigned, postNotice: does the strand-first, program-fallback signing behave as before for the existing notices, and does the new path sign as the session's strand from a non-orch pane?"
  - "internal/loomcli/landingdeps.go and parentreview.go: is the row's own ask still signed as LOOM, and the session's re-ask signed as its own strand, with no path left that posts through PostAsProgram from a session pane?"
  - "The new `undelivered` state: is every consumer of the ask states handled (landingshed prreviewask.go, shedadapters parentseat.go, driftengine judgeparent.go and verify.go, the `parent-review await` verb), and does `no-parent` keep meaning only that no parent orch resolved?"
  - "Survivors at internal/driftengine/judgeparent.go:207 and internal/shedadapters/parentseat.go:226: does a test assert that a refused post ends undelivered rather than no-parent, with the error as reason, in both seats?"
  - "Persisted ask records (.lym/parent-review/asks.json): can an older record or another binary reading it break on the new state value?"
  - "Docs and stencil: does contracts/orchrole/orch-role.md and the autonomous PR-Review landing block tell the session what the issue expects, and do the doc.go edits match the code?"
  - "Test economy: prreviewask_integration_test.go is not judged by the redundancy report; check it against TestPRReviewAsker_Ask for duplicated coverage."
---

Seed focus for the issue #200 fix: a refused post must not read as a missing parent, and the session's re-ask must be delivered.
