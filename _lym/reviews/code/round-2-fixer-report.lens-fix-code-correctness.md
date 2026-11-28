```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the F8 fix changed (lens fix-code-correctness).

no findings

- The change adds `tiergate.Require(t, 0)` as the first statement of `TestRunAutonomyStamps_PostsARefusalOncePerEpisode`, plus the `tiergate` import. No signature or production code changed, so no caller is affected.
- Tier 0 selected: `Require` returns, the test runs `t.Parallel()`, and all four subtests pass in the gate run above.
- Tier 0 not selected: `Require` skips the whole test before any subtest starts.
- Invalid tier selection: `Require` fails the test through `t.Fatal`.
- The defect check holds: the review reports `TestTierTags_OneTierPerTestFile` failing without the call, and it passes with it.

Further tool calls: none.
