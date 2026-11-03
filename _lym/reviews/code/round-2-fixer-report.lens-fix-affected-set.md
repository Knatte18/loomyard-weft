```yaml
lens: fix-affected-set
outcome: reported
```

**Task:** check that the F1 fix covers every stale site in `_lym/landing/summary.md`. This is the lens fix-affected-set. I only checked F1, because it is the only finding.

**Findings:** no findings.

The search covered all of `_lym/landing/` for "unreadable", "unusable", "no rule", "soft", "Warn", "falls back" and "transient". The only hits are the three edited spots (lines 26-28, 43-44 and 57-58). Line 41 ("lazy and soft") is a heading that still holds. Line 56 is a `ResolvePullRequest` row about the field being off, not the rule read, so the new behavior doesn't touch it. Line 46 is about push skip and a missing upstream, which is unrelated.

**Unchecked:** I did not check that the test rows added at lines 57-58 match actual test names. The review's focus answers confirm that rows exist in `prreview_test.go` and `finalize_pr_test.go`.

**Tool calls:**
1. `grep -rnEi` over `_lym/landing/`, to rerun F1's search for the stale rule-read claims across the whole landing directory, which my context didn't show.
