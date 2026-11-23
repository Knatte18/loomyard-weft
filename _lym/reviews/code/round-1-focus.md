---
round: 1
focus:
  - "internal/hardenshed/campaign.go (reviewerRepeat and CheckCampaign): does a one-entry reviewer list pass while two neighbouring equal entries in lists of two or more still refuse, and is the refusal message correct for every length?"
  - "internal/hardenshed/template.yaml and doc.go: do the comments and doc text state sonnet[high] alone as the default and a rotation as the operator's per-run choice under run_settings hardener, with no leftover text describing the rotation as the default?"
  - "Any remaining reference to the opus/fable/sonnet rotation as default elsewhere in the repo (docs, PATTERN.md, pattern/, stencils): does the diff leave one the decision asks to update?"
  - "internal/configreg/runsettings_test.go: does the new row actually prove hardener.reviewer and hardener.fixer are accepted as lists, and does the code already accept them, as the decision requires checking, with no fix needed or missing?"
  - "internal/hardenshed/campaign_test.go and config_test.go: were any assertions widened or removed in tests that exist at the base, and do the tests still check what their names state, including a one-entry list passing?"
---

Seed focus for round 1 covers the three operator decisions (default, per-run choice, list-typed run_settings) and the parent's one-entry-list amendment.
