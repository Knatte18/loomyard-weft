```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F3 change set (the weft replay test's new pre-replay wait count) for consistency with its surroundings.

Findings:

- **LOW**, doc comment of `TestPull_ReplayHeldByAPauseWithoutTheWeftLocks` in `internal/fabricengine/pull_integration_test.go`. The comment disagrees with the constant it introduces.
  - The comment says the admission "is the only wait made while the weft head still sits at the local commit beyond the gated fetch of the records side". That reads as one fetch wait plus the admission, so 2.
  - The constant is `wantWaitsBeforeReplay = 3`. The fixer's own run measured 3 waits with the admission and 2 without it.
  - So the pre-replay gated calls are two, not one, and the comment is stale against the test's own measurement.
  - The error message was already reworded to "the gated calls of the pull before it".
  - A reader who counts fetches from the comment will expect 2, and may "fix" the constant to 2.
  - Suggested wording: say that the pre-replay gated calls of the pull make a fixed number of waits, and the admission adds one more.

I checked the siblings and found no other disagreement:
- `pull.go` is back to its committed state, so `lockWeftForReplay`'s doc ("passes the process's pause gate before it takes the write lock") still matches the code.
- The gate tests for Bolt pull, push retry and takeover were not touched.

Tool calls made: none. My context already held every changed file and the fixer's measurement output.
