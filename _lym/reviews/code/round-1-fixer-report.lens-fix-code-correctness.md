```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1–F9 change set.

Findings:

- **LOW** — `internal/fabricengine/ownerarchive.go`, `archiveOwnerTip`: this function was changed beyond what F8 named. F8 named only `readSideOwnerArchives`.
  - Before, any `RemoteURL` error returned `"", nil`. Now only `gitrepo.ErrRemoteNotConfigured` does, and any other error fails the pair's archive.
  - Take an input where `pathPresent(dir)` is true but `dir` is not a readable git repository, such as a leftover empty records worktree directory. `ArchiveOwnerTips` used to archive nothing there. It now returns `ErrArchiveFailed` for that pair.
  - This follows F8's reasoning, but no test covers it. The existing `ArchiveOwnerTips` rows (torn pair, broken origin) don't reach this branch, and the gate was green.
  - The risk is a stricter archive in adopt's reconcile for an odd pair directory. The fixer should decide whether to keep it and name it in the report.
- **No finding** on the other changes:
  - `RemoteURL` now wraps `ErrRemoteNotConfigured` for an absent remote (via `git.ErrRemoteNotFound`) and for a remote with no URL. The `upstream` row asserts `errors.Is` and passed. Other `RemoteURL` callers only test `err != nil`, so none changed behaviour.
  - Each added test fails with its defect in place:
    - Teardown row: the `archive/root/<tip>` tag is pushed to origin and the row asserts that no `archive/*` tag is fetched locally.
    - Records-branch row: `archiveRecords` with `pushTip` is listed if the records tag is compared against the code branch.
    - `--here` rows: dropping `NoOrch` in `mergeRequest` removes `--no-orch` from the spawn args. Removing the `flags.Here` check in the discard refusal reaches `deps.Discard`, which errors.
    - Integration test: a wrong `Worktree` name changes `subject` or yields `no_subject`.
  - Caveat: the sort test's two code-side tags come from map iteration order, so a missing sort fails it on some runs, not every run. That is a weak detector, not a wrong result.
  - I found no broken caller of a changed signature.

Further tool calls: none. My context already held every changed file.
