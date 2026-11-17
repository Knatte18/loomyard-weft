```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal, error or next step the round-2 fix introduced can be carried out from the state it names.

Ways forward the fix introduced:
1. D5: under `--board-path`, the missing-`board.html` refusal names only `lym board rerender`.
2. D6: the vscode workspace builder refuses a hook member named `folders` or `settings` as a programmer error, which the regeneration returns, writing nothing.
3. D6: fabric refuses a hook member named after the beam file's generated mark in the same way.

Findings:

- **LOW**, D5 second bullet: "Under `--board-path` the refusal names `lym board rerender` alone, since no worktree was resolved."
  - **Summary:** the refusal gives no place to run `rerender` from, so following it can fail or rerender the wrong board.
  - **Evidence:** `rerender` keeps the strict `lymcwd.Resolve` (`internal/boardcli/cli.go:146`). Run from a cwd other than an anchored directory, it fails with `ErrCwdOutsideAnchor`. Run from some other hub's worktree, it renders that hub's board, which need not be the `--board-path` directory.
  - **Precedent:** for this same no-location case, the subtree already names a place to run the verb: "run the verb from a worktree of the hub" (`internal/boardcli/cli.go:75`). The refusal could say the same, for example "`lym board rerender`, run from the anchored directory of a worktree of the hub". It cannot name one specific directory, since no worktree was resolved.
- Items 2 and 3 are refusals for a programmer error, and the way forward is to fix the hook. That is reachable, so no finding.

Tool calls:
- Read `internal/boardcli/cli.go:110-169`: my context lacked the pre-run's `--board-path` branch and how `rerender` resolves its cwd.
- `grep` for `rerender` and `board-path` in `internal/boardcli`: to find whether the subtree already has a way-forward wording for the no-location case. My context had only line 75's location.
