```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (new `TestCommitWeftAt_SeedFailureFailsBeforeStaging` in `commitweftat_test.go`) for agreement with its surroundings.

**Findings**

1. **LOW** — `internal/fabricengine/commitweftat_test.go`, file header (lines 3-11).
   - The header summarizes the file's coverage as "a real commit on a dirty worktree, a no-op on a clean one, and the SkipGit early return."
   - The fix added a fourth behavior, the seed-failure path, and left the header unchanged.
   - A reader deciding from the header whether this file covers the seed-failure path would conclude it does not.
   - The `code-quality` file-header rule requires the header to say what is in the file.
   - Fix: add "a seed-failure abort before staging" to that list.

Everything else agrees:
- The test name follows the sibling `TestCommitWeftAt_<Behavior>` pattern.
- It carries a `//testtiming:keep` directive in the siblings' form.
- It reuses the `newPlainWarpRepo` and `gitStatusPorcelain` helpers.
- Its comment about git rejecting a directory as the exclude file matches the failure the gate run showed.
- The test asserts what the `commitWeftAt` doc comment in `recordsgit.go` says: it seeds before staging, and a seed error returns `("", false, err)`.

Further tool calls: none, because my context already held every file I needed.
