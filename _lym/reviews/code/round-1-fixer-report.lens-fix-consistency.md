```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (new `TestHubLandDeps_HeldSlugs` plus its two imports in `internal/beamcli/deps_integration_test.go`) for agreement with its surroundings.

No findings.

- **File header:** "drives hubLandDeps's seams against a real hub pair and real git" still describes the new test.
- **Sibling setup:** the test uses the same `tiergate.Require(t, 1)`, `t.Parallel()`, `boardkit.InstallReplay`, `hubforge.CopyHub` and `lymcwd.ResolveWorktree` calls as the neighbouring tests, and is placed before them in the order the review asked for. It does not call `t.Setenv`, so `t.Parallel()` is right.
- **Twin at the `HeldSlugs` source:** the test builds its run directory with `shedrun.RunDir(prime, "held-run")`, the same path `shedrun.RunIDs` reads under the anchor. The task branch comes from `git branch` in the prime worktree, which `fabricengine.TaskBranchSlugs` reads as a local branch under the empty prefix. The two sources and the test's two inputs line up.
- **Imports:** `slices` and `shedrun` are both used, and each sits in alphabetical position within its group.
- **Doc comment:** the comment on the test names behaviour only and does not pin a count.
- **Gate run:** the fixer's tier-1 gate run shows the test passing.

Tool calls made: none; my context already held every changed file.
