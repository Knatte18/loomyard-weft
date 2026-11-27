---
round: 3
focus:
  - "internal/landingshed: check how the landing.yaml key describe_review_timeout_min is spelled at each site that names or loads it (the timeout Warn in describereview.go, the ConfigSpec map and positive-value list in config.go, the constants in keynames.go), against PATTERN-config-key-names and PATTERN-code-conventions."
  - "internal/landingshed/prreview.go, syncText and judge: when the pull request text cannot be built, the review's ending is held. Settle whether the message names that hold and a way forward, whether a test reaches it, and whether following the way forward lets an approval or lym loom pr-continue end the review (PATTERN-refusal-way-forward)."
  - "internal/landingshed/describe.go and describereview.go: the writer and reviewer runs start together. Settle how much time the review gets when the writer uses most or all of describe_timeout_min, whether the text can land unreviewed, and whether the timeout comments in template.yaml and config.go and the claims in _lym/landing/summary.md match the code."
  - "internal/beamcli/deps.go hubLandDeps and internal/landingshed ReplaceBeamDescription and RecordBeamDescription: settle whether any test fails if the configured forbidden terms are no longer forwarded or applied."
  - "contracts/stencils/beam/beam-template-describe.md: settle whether the beam describer is told to keep the tooling's names out of the pull request title and body, as the amended board entry asks of the Describe step, comparing with landing-template-describe.md."
---
