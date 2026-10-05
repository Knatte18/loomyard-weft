# Card 4 — fix-and-discussion-stencils

**Prosa:**
- `contracts/stencils/burler/burler-step-3-fix.md`
- `contracts/stencils/loom/loom-template-discussion.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Uses:**
- `contracts/stencils/webster/webster-body-verify-fix.md`

**Intent:** Bound test additions in the Burler fix step (D6) and make Discussion's must-cover scenarios behaviors rather than tests (D7).

- `burler-step-3-fix.md`, `## Fix-everything rule`: add one rule: a fix adds a test only when the finding is a coverage gap; for any other finding it corrects the existing test, per `PATTERN-test-economy`.
  The file has no conditionals and gains no stencil marker.
- `loom-template-discussion.md`, the "Must-cover test scenarios go under `## Acceptance criteria`" bullet: add one sentence: a scenario names a behavior to cover, not a test to write, and an existing test or a new table row may meet it.

`webster-body-verify-fix.md` is read, not changed: its "never delete, skip or weaken a test" rule stays, since removing a redundant test is a review finding's fix, never a way to make a failing gate pass.

`claims_test.go`, `wordingClaims`: add one claim on the new rule's distinctive phrase to the `burler-step-3-fix.md` row, and one on the new sentence's distinctive phrase to the `loom-template-discussion.md` row.
No new test function is added.

**ImpactSummary:** two rows of the existing claims table in `contracts/stencils`; the Burler fix stencil reaches every Burler round and the Discussion stencil reaches Discussion-Write, and no Go production code changes.

**Commit:** 4: fix and discussion stencils: a fix adds a test only for a coverage gap
