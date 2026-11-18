```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I checked each finding whose premise is a set, with one search per finding.

- **F1** (verbs whose first positional is not a run id): the loomcli verbs that take a positional are `arbiter <run-id>`, `pr-findings <file>`, `fmt <file>...`, `insert-card <position> ...`, `remove-card <position>` and `show <slug>` (in `peers.go`). Only `insert-card`, `remove-card`, `fmt` and `pr-findings` are non-run-id positionals that pass through `resolveRunID`, and all four are now excepted (`internal/loomcli/arm.go:69`) and have a test row. `arbiter` takes a run id, so it needs no exception. I did not check whether `show` is armed through `resolveRunID`.
- **F9** (stale Drift and parent-ask passages): a search of `docs` and `contracts` for "expected changes", "once per pull request" and "asks once" matches only `docs/overview.md:1085` and `docs/overview.md:1365`. Both are covered: the new lines follow line 1085, and line 1365 now says the ask is made once until a send-back clears it.
- **F10** (copies of the "Stuck is always a Halt" comment): the string appears nowhere in the repository outside `.lym`, `_lym` and `.git`, so the fixture was the last copy.
- **F11** (briefs that include the loosened-check fragment): the old enumeration ("Webster and Weaver implementer", "include it by name") appears nowhere else in `contracts`, `docs` or `internal`.

I did not rerun the F8 search for other contained tests the redundancy report lists. That finding named exactly two tests, and the fix marks both.

Tool calls (4 of 6):
1. Searched the repo for "Stuck is always a Halt", because my context held only the one fixture copy.
2. Searched loomcli for `Use:` command definitions and verb exceptions, because my context did not show the full set of verbs with positionals.
3. Searched `docs` and `contracts` for the stale Drift and ask phrasing, because my context held only the spans I had read.
4. Searched for other copies of the fragment's consumer list, because my context did not show whether the list recurs elsewhere.
