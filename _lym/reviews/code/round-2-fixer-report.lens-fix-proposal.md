```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposal for F8 (add `tiergate.Require(t, 0)` to the refusal-episode test), the only fix applied as proposed.

no findings

- **Tier 0 holds:** the test drives `runAutonomyStamps` in memory with a stub `post`, with no hub, git or tmux.
- **No mixed tiers:** `internal/beamcli/dobbyautonomystamps_test.go` holds one `Test` function, so the one-tier-per-file rule holds. Other `beamcli` files naming tier 1 are separate files.
- **Placement:** `tiergate.Require` is the first statement, before `t.Parallel()`. Subtests inherit the skip because the parent skips before registering them.
- **Gate result:** the parent's tier-0 run showed `TestTierTags_OneTierPerTestFile` and the test with its four subtests running and passing.

Further tool calls: none.
