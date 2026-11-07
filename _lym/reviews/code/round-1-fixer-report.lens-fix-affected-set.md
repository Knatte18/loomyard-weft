```yaml
lens: fix-affected-set
outcome: reported
```

No findings. All three sites still on the old steps differ from `clashLayer` and were rightly left alone.

Unchecked: nothing. I ran one search per finding.

**F1 (the `clashLayer` set).** I searched for the three-step sequence `setupConflictingDivergence` + `branchAtCurrentHEAD(... "feature-weft")` on a `newMergePairFixture(t, ".")` pair with `clash.txt`. The full list of `setupConflictingDivergence` calls covers every candidate.
- `mergein_recovery_integration_test.go:1208` has a `commitOnWarpBranch("decoy", ...)` step before the conflict, so `clashLayer` does not match.
- `mergelock_integration_test.go:177` and `:244` use `commitOnWeftBranch("feature-weft", "_lym/clean.txt", ...)`, a different weft step.
- `weftguards_integration_test.go:98` and `mergein_integration_test.go:240` use `conflict.txt` (`conflictLayer`'s file), not `clash.txt`, and have a different weft step or none. They were out of F1's scope.

**F2 (stale "Arrange" comment).** `grep` for `Arrange moves|Arrange adds|Arrange creates` across `internal/fabricengine` found no other stale instance.

Tool calls used:
1. `grep` for `setupConflictingDivergence` and the "Arrange moves/adds/creates" phrases, to rerun F1's and F2's searches, because my context had only the `clashLayer` consumers.
2. Read of the candidate sites at `mergein_recovery_integration_test.go:1203-1214`, `mergelock_integration_test.go:172-183` and `:240-250`, `weftguards_integration_test.go:94-100` and `mergein_integration_test.go:236-244`, to compare their steps with `clashLayer`, because my context did not show them.
