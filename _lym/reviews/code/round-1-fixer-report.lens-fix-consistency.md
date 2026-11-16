```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fixes to F1, F2 and F3 for sibling or twin text that no longer agrees with them.

- The remaining prime-only wording sits in `docs/research/speed/**` and `docs/research/speed/testmap-payoff-2026-11-15.md:42`. Those are dated research reports that record one point in time, so they are exempt.
- The twin of the F1 edit in `pattern/PATTERN-test-speed.md` (the "plans every map" line) is changed along with it.
- `internal/gatecli/redundant.go:51` already says "the hub's shared maps from the prime, this worktree's own elsewhere", which agrees with the new `testmap` Long and the PATTERN text.
- The `testmap` Long still says it prints "the plan's and mismatch counts", which the restored audit sentence now explains.
- In the F2 test, the comment, the `anchor` variable and `git add backend` agree with `coverageMapsDir`, which uses `location.AnchorPath()`.

Tool calls:
1. Ran `grep` for prime-only testmap and redundant wording across `pattern`, `docs` and `internal/gatecli`. My context didn't hold the sibling files.
