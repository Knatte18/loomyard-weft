```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes to card 22 and card 25 agree with the cards, criteria and cross-references around them.

no findings

- **Card 22:** after the `compareLockRetry` Delete bullet came out, nothing else in the card names the constant. That includes its Intent, its ImpactSummary and the `compareCmd` bullet. A grep of `_lym/plan/` finds no other card that deletes or edits the constant.
- **Card 25:** the rewritten `accepted.go` bullet keeps `testFiles` for `Check`'s in-flight `partial` scope. This matches the card's own `records.go` bullet, which says "the `partial` scope stay[s] readable for the verdict and D3's in-flight rule". It also matches `internal/driftengine/check.go:247-274`, where `Check` calls `skippedCases`, which calls `testFiles`. No card removes `skippedCases`. The ImpactSummary has no claim that depends on `testFiles`.
- **`accepted.go` file header:** after card 25 the header still describes "the accepted profile". It was already stale before the fix, when the card kept only `copyFile`, and the fix did not make it worse. The rubric excludes such a gap, so it is not a finding.

Tool calls:
- One grep of `_lym/plan/` and `_lym/*.md` for `compareLockRetry`, `LockRetry`, `testFiles`, `accepted.go`, `skippedCases` and `in-flight`. My context showed only cards 22 and 25, not the other cards or the overview.
