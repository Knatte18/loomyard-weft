---
round: 2
focus:
  - "In `markedScope` (internal/gatecli/only.go) and the guard loop of `planTierRuns` (internal/gatecli/tiers.go): when does a marked test in a file of a tier declared `manual: true` run? Check the case where the key's tiers list that tier, the case where they do not, and the case where they list only a tier that covers it."
  - "In the marked case of `TestGateTest_Scopes` (internal/gatecli/scopes_integration_test.go): would the assertions fail if a tier1 marked test ran under tier0's command, env or slots, and is the manual-tier behaviour of marked tests covered?"
  - "Do internal/gatecli/doc.go, internal/gateslot/template.yaml, pattern/PATTERN-test-speed.md, the refusal texts in internal/gatecli/tiers.go and _lym/landing/summary.md each describe which marked tests run, which are refused and which files changed as the diff has them?"
---

The first entry covers the one gating concern of the judged round; the others cover the text and tests that a change to it would touch.
