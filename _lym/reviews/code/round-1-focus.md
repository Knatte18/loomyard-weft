---
round: 1
focus:
  - "internal/loomshed/redfirstgate.go verify: does the exit-5 pass branch hold for every caller, including the final evaluation and a resumed hold, and does a bug fix with exit 5 pass even when the diff adds no test?"
  - "docs/overview.md lines near 997 and 1113 still describe a check that cannot judge the tests as holding the handoff; the diff did not touch them. Check whether the repository's rules require that doc to land in the same diff."
  - "internal/gatecli/redfirst.go: the verb's own help text and the doc comment near line 105 against the changed refusal message and gatecli/doc.go. Do they agree on what exit 5 means to a caller?"
  - "internal/gatecli/doc.go line 489 list of exit codes and contracts/stencils/darn/darn-template-write.md: do they carry the pass wording the board entry expects, with no leftover hold or waiver wording for exit 5?"
  - "Tests in internal/loomshed/redfirstgate_test.go: the hold test switched its exit code from 5 to 4; does it still check what its name says. Does any test cover exit 5 together with TimedOut or RunnerCrash, which the new branch excludes?"
  - "Commit message Root-Cause trailer against the diff: does the named defect, holdsForCommitOrRecord counting exit 5 as a hold, match what the diff removes?"
---

Seed focus from reading the diff against the board entry in the decision record.
The record has no decisions or acceptance criteria beyond the Goal, so the Goal is the measuring stick.
