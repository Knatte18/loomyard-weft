```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix against the sets in the findings (F7 stale references, F5 role-label sites, F2 `fills` callers).

No findings.

- F7: no `NewPreflightWithBatcher` remains in any `.go` or `.md` file outside `.lym` and `_lym`.
- F5: the only `seat %q` errors that still quote `seat.Name` are `table.go:280` and `table.go:283`. The review rejected both on purpose, since they quote the name under validation. Every other site uses the role label.
- F2: `fills` has one caller, `CheckMarkers` at `prompt.go:138`, so the blank-value change reaches no other site.

Tool calls:
1. Grep for `NewPreflightWithBatcher` (F7): my context did not hold a repo-wide symbol search.
2. Grep for `seat %q` errors quoting `seat.Name` (F5): my context did not hold the full set of error sites.
3. Grep for `.fills(` callers (F2): my context did not hold the full set of callers.

(All three ran in one Bash call.)
