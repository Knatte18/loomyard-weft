```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 and F2, both fixed as proposed.

No findings.

- **F1 (`GitDirsThroughLink` row):** I built the case where each `EvalSymlinks` call in `readCheckout` is dropped.
  - Dropping the `geometry.GitDir` call leaves `gitDir` as `<base>/main-link/.git/worktrees/wt`. The row expects `<base>/main/.git/worktrees/wt`, so the `DeepEqual` on the launch fails.
  - Dropping the `geometry.CommonDir` call leaves `commonDir` as `<base>/main-link/.git`. The row expects `<base>/main/.git`, so it fails the same way.
  - `dotgit` does no symlink resolution (the grep in the parent's context shows none). `commondir` `../..` is joined lexically from the linked `gitdir:`, so both paths really do arrive unresolved.
  - `mainRoot` resolves to `repo.mainRoot` through its own `EvalSymlinks` call, so the no-warnings assertion still holds.
  - `wantGitDirs` takes its linked-worktree branch because `anchor/.git` is a file, so `isDir` is false.
  - The `tt.gitDirsThroughLink` block sits in the row loop and runs before `newEngine`, and the row sets no records-checkout fields, so it collides with no other row.
- **F2 (`TestPrepare_ExpandsHomeInWritableRoots`):** If the `~/` expansion were dropped, `expandHomePath` would refuse `~/cache` as relative, so `Prepare` would fail the test. If it expanded but skipped the symlink resolve, the `cache` root would differ from the expected line.
  - `[~/cache]` parses as the string `~/cache`, since only a bare `~` is YAML null.
  - `cache` sits under `repo.base`, which `newRepoFixture` already resolves, so the expected path equals the resolved one.
  - It sets both `HOME` and `USERPROFILE`, so `os.UserHomeDir` is covered on Unix and Windows.
  - It omits `t.Parallel()` and a comment above it gives the environment as the reason, as the review asked.
  - The gate run in the parent's context passed this package with no failures.

Further tool calls: none.
