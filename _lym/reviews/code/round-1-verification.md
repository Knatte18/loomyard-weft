---
round: 1
entries:
  - key: batch-gate-key-not-one-constant
    status: closed
    finding: F1
    reason: "e4f5e2397 uses gateslot.BatchGateKey in tiers.go goReadGates, runplan/cardgate.go, weaverengine/strand.go (gateKeyBatch deleted, both uses) and webstercli/beginbatch.go; stopKindBatch is left alone"
---
