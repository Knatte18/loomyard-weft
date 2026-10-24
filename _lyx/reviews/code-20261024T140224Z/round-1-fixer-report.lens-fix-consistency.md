```yaml
lens: fix-consistency
outcome: reported
```

no findings

The only occurrence of either ID outside the excluded `docs/research/` and `docs/benchmarks/` trees is the changed line `internal/discussionparser/validate_test.go:258`. No twin and no assertion keys on the old ID, so the fixture row has no stale sibling. The row still has the `- `PATTERN-<id>` — changes: <text>` shape, so it still exercises a changes action.

Out of scope, no finding: line 257 uses `PATTERN-cwd-resolution`, which the review did not flag. I did not check whether it is a removed ID.

Tool calls:
1. `grep` for `hub-suffix` and `old-hub-rule` across `.go`, `.md` and `.yaml`: my context lacked a repo-wide search for twins of the changed ID.
