---
round: 2
focus:
  - "internal/testcover/runner.go and internal/gatecli/testmappackage.go: settle whether every `go` invocation the testmap verb starts for a package runs with its working directory in the module root the verb resolved, including a module below the worktree root."
  - "internal/gatecli/testmappackage.go and internal/gatecli/prebuild.go: settle whether a terminating signal during the lym build, a coverage build or a test is kept out of the package's failure map, so the package is retried at the same HEAD, and whether the call context reaches those child processes."
  - "internal/gatecli/prebuild.go: settle whether the lym build is bounded after its group kill when a descendant outside the group holds the output pipe."
  - "internal/gatecli/testmap.go mapPackages: settle whether a named package with no Go file tracked at HEAD is listed as skipped with its cause."
  - "internal/gatecli/testmap_integration_test.go and internal/testcover/runner_test.go: settle whether staleness by Go file set and by file content, the build timeout, the grace bound with a descendant holding the output pipe, and the refusals (no hub, no mapped tier, unknown package) each have a test that would fail if the behaviour broke."
---

Focus targets the round 1 findings that remain open; nothing here restates a verdict.
