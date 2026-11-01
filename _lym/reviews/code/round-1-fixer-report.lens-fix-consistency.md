```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set (commit c66eebf8, `tools/boltrename`) for agreement with its surroundings, through the fix-consistency lens.

Findings:

1. **LOW**, `main.go` (`main`, `inspectHub` called with the resolved `hubPath`).
   - **Summary:** the checkout's git link checks now compare against the symlink-resolved path, while the files they read hold whatever path git recorded.
   - **Evidence:** `inspectHub` expects `.git` to read `"gitdir: " + h.adminDir(oldName)`, and the admin `gitdir` file to read `filepath.Join(hubPath, oldName, ".git")`. Both are built from the `hubPath` argument, which is now `EvalSymlinks` output. A hub whose worktree links were written through the symlinked path no longer matches and stops with "links are not as boltrename expects". That fails closed and the message names the path, so it is not a wrong result. `renameBolt` also writes the resolved path into the new links. The header doc, which says it stops when "the checkout's git worktree links are not the ones it expects", does not mention this.

2. **LOW**, process only: the fixer committed F1 and F2 together as `c66eebf8`. `instruction-fix.md` asks for one commit per finding. The code is consistent, so only the commit-discipline sibling disagrees.

Checked and consistent:
- The `liveProcesses(hubPath, absPath, processes)` signature has one caller, `main`, and the test, both updated.
- The doc comment matches the new parameters.
- The package header's "refuses while a lym process, or any process working in the hub, is alive" still holds.
- New `TestInspectHub_StopsOnWhatItDoesNotExpect` rows follow the existing naming and `wantPart` style.
- Each new row's `wantPart` matches the branch's message text: `notes.txt`, `no prime with its weft`, `reads "gitdir: /elsewhere"`, and `_bolt exists`.
- The own-PID row would be listed if the skip at `main.go:76` were removed, so it pins that skip.
- The `/link/...` command-line row exercises the `absPath` test, and `900000002`'s cwd exercises the resolved-path test.

Tool calls: none beyond what my context held.
