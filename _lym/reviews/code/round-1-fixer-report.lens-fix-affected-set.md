```yaml
lens: fix-affected-set
outcome: reported
```

no findings

The sets behind the two findings are covered:
- **F4:** "take CopyHub" now appears once, at `internal/hubforge/template.go:357`, and the fix already names `CopyRegistered` there. No other source or doc carries the old advice.
- **F3:** the once-only-registration rule now sits in `internal/hubforge/template.go:75`, `internal/hubforge/doc.go:57` and `pattern/PATTERN-test-isolation.md:202-203`. The three agree.
- **F1:** the `Skip`, `Skipf` and `SkipNow` rows cover every skip method that `templateBuildTB` refuses.
- **F2:** one `Errorf` row is what the review asked for. `Error` and `Fail` share the same `record` path.

Unchecked: none.

Tool calls:
1. `grep -rn` for "take CopyHub", "once-only registration" and "Bolt replay" over `*.go` and `*.md`. It answered both set searches (F3 and F4) in one call; my context had no repo-wide view of these strings.
