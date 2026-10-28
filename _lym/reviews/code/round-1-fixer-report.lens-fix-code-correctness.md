```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 and F2 test additions through the fix-code-correctness lens.

No findings.

- **F2 subtest** (`starts_test.go`, "the limit is read right before each start…"): I ran it step by step with `maxRuns` 3 and tasks a and b.
  - For a, the first read is `free`, so headroom is 1800, which is not below 500. The task isn't held, "start a" is logged, and `running` becomes 1.
  - For b, the second read is `near`, so headroom is 3, which is below 500. The hold line is logged and the loop breaks, so only a spawns.
  - The expected log lines `["start a", "hold starts: 2997 of 3000 …, below task_headroom 500"]` match `taskLimitHolds` at `starts.go:123`.
  - With the defect in place, a version that read the limit once per tick would see only `free`. It would start b too and log no hold, so both assertions fail.
  - `reads` is a closure variable local to the subtest, so parallel subtests don't share it.
  - The `free` and `near` variables are shared read-only structs, so that is safe.
  - The gate run for `./internal/dobbyengine` passed.
- **F1 assertions** (`dobbydeps_integration_test.go`): `beamDeps.TaskLimit` and `primeDeps.TaskLimit` are checked against nil and never called, so the host's real cgroup is not read.
  - Dropping `TaskLimit: proc.ReadTaskLimit` from `hubDobbyDeps` makes both assertions fail.
  - The tier1 run for `./internal/beamcli` passed.
  - No caller or signature changed, since the fix only adds test code.

Further tool calls: none.
