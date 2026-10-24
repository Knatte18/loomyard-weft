```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F1 is the only finding. Its set is live files outside `docs/research/`, `docs/benchmarks/`, `testdata/` and `_lyx/` that cite `PATTERN-hub-suffix`. The rerun search finds no remaining occurrence, so the fix covers the whole set.

I checked only `PATTERN-hub-suffix`, the one ID the finding named. I did not search other removed IDs, so I left no other finding unchecked.

Tool calls:
1. `grep -rn "PATTERN-hub-suffix"` over the worktree, excluding the out-of-scope directories. This reran F1's own search. My context held only the single line the fixer had changed, not the whole tree.
