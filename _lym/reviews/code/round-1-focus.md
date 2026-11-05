---
round: 1
focus:
  - "Scope against the board entry (`## Goal`): the entry asks to find the test or code that creates `.git` under the shared temp directory, and to add a guard. Settle from the diff and commit message whether the source hunt was done or left open, and whether the guard alone meets the entry's second bullet."
  - "`internal/testkit/tmuxkit/tmuxkit.go` `checkNewGitEntries`: settle whether a `.git` appearing in a shared directory and removed by an empty-directory `os.Remove` can race with another concurrent package binary, and whether a `.git` file (a worktree pointer) or a symlink is handled sensibly by the `IsDir` and `Lstat` branches."
  - "`Main` ordering on Linux and Windows: settle whether the snapshot is taken before `setEnv`/`setTempEnv` and the run directory creation so it sees the right shared directories, and whether the early-return paths (socket path check failure) skip or break the new check."
  - "`sharedTempDirs`: settle whether `filepath.Dir(dir)` and its parent are the lym temp root and the system temp directory in every case `lymtemp` can produce (override env, Windows, macOS symlinked temp), and what happens when the two coincide or the root is not under the system temp."
  - "`internal/testkit/tmuxkit/tmuxkit_test.go`: settle whether each new test covers behavior not already covered by another test in the same file (test economy), whether any test writes outside its own `t.TempDir()` (the rule the change enforces), and whether the removal-by-hand and non-directory branches have coverage."
  - "`pattern/PATTERN-test-isolation.md` and the `tmuxkit.go` package doc: settle whether the new bullets match what `Main` does (exit code, removal, bounds) and follow the repository's comment and PATTERN conventions; check the commit message and any summary claim against the diff."
---

Seed for round 1. The diff is one commit touching `tmuxkit.go`, its test, and one PATTERN file; the decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
