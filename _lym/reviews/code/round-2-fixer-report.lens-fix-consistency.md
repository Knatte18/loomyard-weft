```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F8 change (`tiergate.Require(t, 0)` and its import in `internal/beamcli/dobbyautonomystamps_test.go`).

no findings

- The call is the test's first statement, before `t.Parallel()`, as the sibling `beamcli` tier-0 tests and F8's `fix` have it.
- The file header and the test's doc comment still hold, and the file has one test, so it names one tier.
- For a twin left unfixed: `TestTierTags_OneTierPerTestFile` passed on the tree after the edit, so it reports no other test missing its tier.

Further tool calls: none.
