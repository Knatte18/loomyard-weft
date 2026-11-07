```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: for each finding whose premise is a set (tests, callers or sites under one key), rerun its search and check the fix covers every member.

- **F1:** the premise is one site, the swallow-and-log path of `renderHeddleOverview`, whose only call is `walk.go:264`. The new row "heddle store that does not load" covers it. It asserts that Ensure returns nil, that HEDDLE.md is left as it was, that the landing config is committed, and that `RenderOverview` still errors. The fix covers the whole set.
- **F2:** the premise is one branch, the `changed == false` branch at `walk.go:264`. The new row "overview already current" covers it. Its assertions are that the commit omits HEDDLE.md, that the commit includes the landing config, and that `RenderOverview` reports `(false, nil)`. The fix covers the whole set.
- **Other `RenderOverview` callers:** the grep found `heddlecli/verbs.go:444`, `heddleengine/replayer.go:108` and the heddleengine and fabricengine tests. None is a walk call site, and the diff does not change them.

Findings left unchecked: none.

Tool calls:
1. `grep` for `RenderOverview` and `renderHeddleOverview` across `internal`: to list every caller, which my context did not hold.
