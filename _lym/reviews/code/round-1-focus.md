---
round: 1
focus:
  - "internal/dobbyengine/describe.go (describeStep, settleDescribe, endDescribe, stopDescribeStrand): does a failed rewrite, a refused record or a timeout leave the recorded description standing and retry at the next landing, as the Design bullet asks? Check the busy flag and the one-notice-per-failure path for double starts or lost landings."
  - "internal/dobbyengine/describe.go mutation survivors at lines 158, 166 and 282: does a test assert the behavior each mutant changes, or is each unreachable by a reasonable test?"
  - "internal/dobbyengine/landnotice.go and contracts/orchrole/role-beams.md: the landed line drops its follow-up, and no orch is asked. Does any remaining text, stencil or doc still tell the orch to extend the description and run lym beam describe after a landing?"
  - "internal/landingshed/beamdescribe.go ReplaceBeamDescription: validation before write, restore of the prior summary.md when the commit fails, and no partial state on a draft with a finding."
  - "internal/beamcli/describe.go and dobbydeps.go: the draft-path record flow, judgeDescribable and the describe gate; the hand path (orch and operator editing then recording) must still work. beamDescribeDeps, beamDescribeFacts and hubDobbyDeps have no mapped test reach: is their behavior covered?"
  - "contracts/stencils/beam/beam-template-describe.md: does the describer prompt rewrite whole, state the beam's end state rather than the way there, and list every member pull request, as the Seen/Design text requires?"
  - "Repository rules: docs/overview.md, internal/dobbyengine/doc.go and internal/beamcli/doc.go land with the behavior change; commit trailers (Pattern-Departure, Loosened-Check) against changed existing tests in land_test.go, landnotice_test.go and deps_integration_test.go."
  - "Test economy across describe_test.go files and dobbydeps_test.go: tautological expectations, duplicated coverage, and tests that assert stencil prose."
---

Decision record holds the board entry alone, with no amendments. The change description at `_lym/landing/summary.md` is a claim to check against the diff.
