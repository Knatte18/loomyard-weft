```yaml
lens: fix-code-correctness
outcome: reported
```

Task: the fix-code-correctness lens over the F1 and F2 code change in `internal/battencli/lifecycle_integration_test.go`.

no findings

What I traced:
- **New top-level test, normal path:** `tiergate.Require(t, 1)` gates it, `posixOnly` returns the step, `CopyHub` builds a fresh hub, and the step runs on it. A `CopyHub` or fixture failure is fatal to this test alone. The tier1 run in context shows it PASS on a fresh hub, so it needs nothing an earlier `CommandRows` step left behind.
- **Windows:** `posixOnly` returns nil and the test returns before building a hub, as the review's fix asked.
- **Serial execution:** the test has no `t.Parallel`, so the parallel tests are not running while `logcapture` holds the logger.
- **Remaining `CommandRows` steps:** the five left passed in the same run. The search found no other `logcapture`, setenv, chdir or worktree-seam use inside a `Rows` step, so no parallel group still takes over process-global state that these patterns cover.
- **F2:** it changes only a doc-comment line, with no behaviour to trace.

No assertion changed, and the fix added no coverage test that would need to fail with the defect in place.

Tool calls (1 of 6):
- Grep of `internal/battencli/*_test.go` for process-global state calls: my context held a `logcapture` search of `lifecycle_integration_test.go` only, not of the other files that define `Rows` steps.
