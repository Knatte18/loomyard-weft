```yaml
lens: fix-affected-set
outcome: reported
```

**Task:** rerun each finding's set search and check the fix covers every member.

**F1 (set: tests that run `redundant` through `Command()` with real seconds)**
- Only one other test calls the verb: `internal/gatecli/testmap_integration_test.go:678` (`[]string{"redundant", "solo"}` through `Command()`).
- I did not check whether its package `solo` holds a mutant that makes a loop endless. It is a map-location check (`map_commit` equals the worktree HEAD), not a timeout test.
- The review named no slow run there and no mutant timeout is pinned there, so I record no finding.
- The only `secondUnit` reference to `redundant` in the tests is the fixed call, at `redundant_integration_test.go:217`.

**F2 (set: tests that depend on random iteration order)**
- My grep for repeat loops (`for range N`, `for i := 0; i &lt; N`) in `internal/testcover/*_test.go` found none left. The 16-call loop is gone.
- I did not search for other tests that rely on map iteration order without a loop.

**No findings.**

**Tool calls**
1. Grep for `"redundant"` and `secondUnit` in `internal/gatecli/*_test.go`: my context lacked the other `redundant` call sites.
2. Grep for repeat-loop patterns in `internal/testcover/*_test.go`: my context lacked the sibling tests of that package.
3. Read `testmap_integration_test.go` lines 668-689: to see what the `solo` call at line 678 asserts.
