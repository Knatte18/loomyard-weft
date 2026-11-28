---
round: 2
entries:
  - key: refusal-episode-test-tier-unnamed
    status: closed
    finding: F8
    reason: "14d45ead6 adds tiergate.Require(t, 0) before t.Parallel() and imports tiergate in internal/beamcli/dobbyautonomystamps_test.go; TestTierTags_OneTierPerTestFile passes and go vet ./internal/beamcli is clean."
---
