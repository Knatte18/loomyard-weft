```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens on the F8 fix, checking that it covers every member of the finding's set.

List of findings left unchecked: none. F8 was checked, and F9 is decision class, so it is not fixed here.

no findings

The five test functions the branch adds each open with `tiergate.Require` before `t.Parallel()`:
- `TestRunAutonomyStamps_PostsARefusalOncePerEpisode`, fixed in `14d45ead6`
- `TestAutonomyLevels_EveryCheckingPathTakesTheThreeLevels`
- `TestPawlDeps_AutonomousReadsUnattendedAsOn`
- `TestOrchAutonomy_KeepsEachLevel`
- `TestWireProduction_OrchAutonomousReadsUnattendedAsOn`

The parent's gate run of `TestTierTags_OneTierPerTestFile` also passed on this tree.

Tool calls (1 of 6):
- `git diff main...HEAD -U3 -- '*_test.go'`, filtered to test function headers and their opening lines. My context held only the diffstat, not the openings of the added tests.
