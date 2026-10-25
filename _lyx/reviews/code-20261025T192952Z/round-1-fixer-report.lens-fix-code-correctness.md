```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check correctness of the F1 `poll` change, the `commandOf` split and the two added tests.

**no findings**

- **`poll` (`overview.go`):** I traced a run that goes Run-Shed, then Plan, then Run-Shed again.
  - The delete runs at the Plan read, which covers both "status not found" and "producer is not Run-Shed".
  - The next Run-Shed read calls `ResolveWorktree` again and gets the new marker's anchor.
  - A resolve error is not cached, because `r.pairs[slug]` is only set after success.
  - The stat-based delete and `childPair` stubs are unchanged, so `TestDobbyRunCells_ReadsTheRunsFromDisk` still holds, and it passed in the gate.
- **F1 test fails with the defect in place:** without the `else` delete, the pair stays at anchor "." and the child status path is the old one.
  - The old file's mtime (100 min ago) differs from the new status's (45 min ago), but the cache polls the old path, so it returns "Plan-Write" instead of "Review".
  - The distinct mtimes matter because the status cache is keyed on mtime alone, not on path.
  - A moved anchor with an identical mtime would also read stale. That is out of scope here, because a new run writes a fresh status file.
- **`commandOf` (`cli.go`):** `Command()` returns `commandOf(&orchCLI{})`, which is identical to the old behaviour. `RunCLIIn` and every other caller still go through `Command()`.
- **F2 test (`runstitle_integration_test.go`) fails with the defect in place:** a wrong worktree name or beam flag at the wiring line changes `c.runsWindowTitle`, and the exact-string assertion catches it.
  - If the pre-run failed before the assignment, the title would stay empty and the test would fail loudly rather than pass.
  - `status` does not start reed (`reedUp` is only called from `start.go:464`), so the test spawns no daemon.
  - The two subtests share one hub but cwd differs (prime anchor versus beam anchor), so parallel runs do not collide.
  - The gate passed with 35/35 on tier 1.

Tool calls made:
1. Read `status.go` (grep and sed of `statusCmd`) to confirm `status` does not call `reedUp`, since my context did not hold `status.go`.
