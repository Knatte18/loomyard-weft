```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F1's premise is a single constant, so the set is every site that names or restates the bound. The search covered `internal/battencli` and `pattern`:
- `startCostBound` appears only at its definition (`internal/battencli/lifecycle_integration_test.go:1417`), the step's doc comment (`:1419`) and the comparison and error message (`:1467-1468`). The comparison and message use the constant, so neither carries a literal 2 s.
- The `testtiming:keep` marker on the rows test (`:313`) names the bound's subject and carries no number, so the reviewer's note that it needs no change holds.
- No other `waits seconds` or `2 * time.Second` wording remains in the package. The one "two seconds" hit, `pattern/PATTERN-test-isolation.md:80`, concerns an unrelated process-exit wait.

Calls made:
1. `grep -rnE` for `startCostBound`, `testtiming:keep`, `waits seconds`, `2 * time.Second` and `two seconds` over `internal/battencli` and `pattern`. My context held only the one file's edited region, so it did not show other sites.
