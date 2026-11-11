```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 fix commits (`a115887b8`, `30c5e8592`, `4b97a9946`, `9acf412c8`).

No findings.

- The code branch target sets `codeWorktree: worktree` (`originreconcile.go:717`), so the `lymVerbDir(b.CodeWorktree)` change in `BranchRewritten` matches the old `b.Worktree` behaviour for code branches.
- The `lym fabric commit`, `anchor`, `push` and `pull` way forwards the fix converted all go through `lymVerbDir` now. The unconverted ones left are in `addparent.go:66` and `addparent.go:72`, in `addadoptonly.go:70` and in `anchortarget.go:146`. Review finding F5 explicitly scopes the add-time ones out, and `anchortarget.go:146` names no directory.
- The `doc.go` sentence now says "failed-push notice or a reconcile stop", which agrees with the code. The following lines (`git` steps at the root, anchor read at build time) still hold.
- The `upstreamAnchorStop` doc comment now describes `codeWorktree` as a root, which matches both callers. Both pass the worktree root (`pull.go:199`, `pushclassified.go:647`).

I made 3 further tool calls:
1. Grepped for other `lym fabric ... in %s` way forwards, to look for unconverted siblings.
2. Grepped for `CodeWorktree` assignments in the reconcile files, to confirm it is set for `BranchRewritten`.
3. Read `originreconcile.go:195-203` and `880-895`, to confirm `CodeWorktree` is set when the outcome is built. The context lacked these lines.
