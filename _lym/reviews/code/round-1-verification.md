---
round: 1
entries:
  - key: config-keys-bare-and-literal
    status: closed
    finding: F1
    reason: "8edf8659d and 5d1b67e9e: keynames.go declares gateHoldFactorKey and gateHoldMinKey, ConfigSpec and both LoadConfig refusals use them, and the named comment, doc and template sites write the keys in full"
  - key: tests-hand-write-module-config
    status: closed
    finding: F2
    reason: "367a1d5ad: TestDobbyGateHolders and TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer write config through configengine.Set; the load failure comes from removing gate.yaml and the error assertion stays"
---
