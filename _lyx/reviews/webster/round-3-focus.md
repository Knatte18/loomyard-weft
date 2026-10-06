---
round: 3
focus:
  - "Comment line breaks: in every `_test.go` comment the branch adds or changes, including doc comments and file headers, does any sentence break mid-phrase or does any line hold more than one sentence, against `PATTERN-comment-line-breaks`? Start with `tools/godocreflow/reflow_test.go` and the comments added in the latest commits of the branch."
  - "Serial tests: for every test or table in the packages the prune touched that does not call `t.Parallel`, does a test, file-header or table comment name the global state or ordering that forbids it, against `PATTERN-test-speed`? Start with `internal/selfreportengine/selfreport_test.go`, whose helper `installGitHubClient` swaps the package-level `NewGitHubClient`."
  - "Parallel tables: for every table-driven test the prune wrote or folded, does the top-level test call `t.Parallel` when its rows touch nothing but `t.TempDir()` or other per-test state, and does each subtest call it too? Start with `TestSeed_WritesExpectedStatus` in `internal/loomshed/seed_test.go`."
---

The three entries restate the sweeps that matter for the next round, each as a question to settle over the whole range rather than a single site.
