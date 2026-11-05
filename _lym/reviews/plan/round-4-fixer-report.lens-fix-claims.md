```yaml
lens: fix-claims
outcome: reported
```

The fix-claims check found nothing wrong.

no findings

Card 25's rewritten Intent bullet makes three claims, and each one holds at HEAD:
- **"keeps only `copyFile` … and `testFiles`":** `accepted.go` declares seven functions: `testFiles`, `requireBaselineTest`, `noBaselineTestRefusal`, `keepAcceptedOutput`, `dropAcceptedOutput`, `buildAcceptedProfile` and `copyFile`. Card 25's Delete list removes the other five.
- **"`copyFile`, which the judge snapshot uses":** `internal/driftengine/judgesnapshot.go:114` calls it.
- **"`testFiles`, which `Check` uses to leave a skipped test's cases out of an in-flight `partial` scope's diff":** `Check` (`check.go:88`) calls `classifyCurrent` at `check.go:166`. That function calls `skippedCases` only when `scope.Scope == ScopePartial` (`check.go:247-248`), and passes the result to `DiffCaptures` as `leaveOut`. `skippedCases` calls `testFiles` at `check.go:274`.

Card 22's change removes one Delete bullet and adds no claim.

Tool calls:
- grep of the `func` declarations in `accepted.go` and `check.go`, plus `copyFile` callers: my context showed only the first 40 lines of `accepted.go` and lines 240-299 of `check.go`.
- grep of the `classifyCurrent` callers: my context did not show what calls `classifyCurrent`, which I needed to confirm `Check` reaches `testFiles`.
