---
round: 2
focus:
  - "internal/landingshed/describereview.go (describeReview, describeReviewSeat, check), landing-template-describe-review.md and internal/shedrecipe's describeEntry: does the reviewer read only a description that passed the mechanical gate, does its time budget cover a writer using its whole budget, and are findings sent once across a re-attach? Check any marker file for removal at a fresh spawn, and check the claims in _lym/landing/summary.md and template.yaml against the code."
  - "internal/loomcli/validate.go and internal/landingshed's config and key-name code: does every new message name a config key through the key-name helper, does each new refusal name a way forward, and does a test reach it and assert the message?"
  - "Tests of the forbidden-term path (the shedrecipe description gate through Env.DescriptionTerms, lym loom validate-description, the beam describer and recorders) and of describeEntry's gate order: would each test fail if the forwarding or the order were dropped?"
  - "internal/shuttleengine/hold.go: what does a held turn end of an operator-started turn with missing output files do for the agent and the operator, and can any reminder or notice repeat, block or misfire on other held turns?"
  - "internal/landingshed/prreview.go (wake, judge, syncText) and Finalize's setRecordedText: can a round end while the pull request's text differs from the description, and can a wait added for that never end?"
  - "Stencils of steps that write or review repository files and the change description, including loom-rubric-code-review.md and darn-template-write.md: do they carry the rule that such text names nothing of the writing tooling that the repository does not itself hold, as the amended board entry asks, without contradicting a rule already there?"
---
