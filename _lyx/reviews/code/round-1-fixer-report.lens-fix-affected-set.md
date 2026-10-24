```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run F1's search (rows crediting a removal to a naming card, and the crediting code) and check the fix covers every member.

F1 is my only finding, and it is a test-coverage gap, not a set premise. I ran its search once.

**Findings**

No findings.

- The set is the rows of `TestNewPlanGate` that credit a `removes` entry to a naming card. Those are `gates_test.go:701` (one card names it), `:702` (the new row, two cards name it), `:703` (only a later card names it) and the existing no-card-names row.
- The set is now complete for the rule in `patternplan.go:129`. `slices.IndexFunc` picks the first naming card, and the rows cover first, later-only, none, and now both.
- The new row catches the "last naming card" mutation. Card one's growth would then go unoffset and `pattern-card-headroom` would fire, but the row expects no finding.
- The review also says crediting "every naming card" fails the row. It would not: card one is still offset, and the second removal of an entry already dropped from `running` changes nothing. That mutation is equivalent to the current code, so no row can pin it. The fix doesn't need to cover it.

**Tool calls**
1. `grep` for `IndexFunc`, `CardNamesPatternEntry` and `namesB`. I needed every site of the crediting rule and every row using `namesB`, and my context lacked the full row list from the changed file.
2. `Read` of `patternplan.go:110-160`. My context did not hold `cardHeadroomFindings`, which I needed to check what crediting to the last or to every card would do.
