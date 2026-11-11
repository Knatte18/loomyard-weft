```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F3, F4, F5 and F6 as the fixer carried them out (commits 03c24559, db1da40e, 17393100).

**Findings**

- **LOW, `internal/fabricengine/retiredlaunchers_test.go`, F5 fix (`run.cmd` added to `fabricWrittenTree` and `fabricWrittenRemoved`).**
  - The proposal pins only one of the four `.cmd` names in `retiredLauncherFileNames`.
  - The F5 premise is "dropping them from the list fails nothing". Dropping `ide.cmd`, `fabric-checkout.cmd` or `ide-menu.cmd` from `retiredLauncherFileNames` (`retiredlaunchers.go:22-27`) still fails no row.
  - Only removing `run.cmd` is now caught.
  - The change is correct but leaves three of the four names unpinned. Adding `ide.cmd`, `fabric-checkout.cmd` and `ide-menu.cmd` to the seeded tree and the removed list would close it.

**Challenged and found sound**

- **F3 (row `SymlinkAtTheContainerIsKeptAndNotFollowed`):**
  - With `Lstat` replaced by `Stat`, `os.Root.Stat` rejects a link that escapes the root. The sweep would then return an error, and the row's `err != nil` check fails.
  - So the row does fail with the defect in place, though through an error rather than a deletion.
  - The harness now seeds `ide.sh` outside the hub and asserts it untouched.
- **F4:** `containmentFailure` is the surviving function at `ancestors.go:21`. The `destroy_test.go` comment about `filepath.Rel` matches it.
- **F6:** `retiredlaunchers.go` still uses `fmt` and `filepath`, and `hubworkspace.go` already imported both. Tier 0 passed for `internal/fabricengine`, so the move broke no caller.

**Further tool calls:** none. My context already held every changed file and the gate output.
