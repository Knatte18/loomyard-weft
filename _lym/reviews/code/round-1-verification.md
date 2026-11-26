---
round: 1
entries:
  - key: tiers-key-path-literal-in-refusals
    status: closed
    finding: F1
    reason: "2ea18609f adds tiersKey and the helpers in both keynames.go files (gatecli tierKey, gateslot tierKeyName) and uses them in planTierRuns' no-tier0 refusal, Config.Tier and Config.Gate"
---
