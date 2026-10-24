```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 and F2 fixes cover every member of their sets (stale ide-task/keybinding descriptions, and comments in the changed test that name another symbol).

Unchecked list: none. Both findings were searched.

**No findings.**

- F1 (set: places that still describe the old ide behavior): a grep across `.md`, `.go` and `.yaml` outside `_lyx/` for "Run Task", "Alt keys", "terminal-only" and "Alt-keys-only" found only two stale sites, `docs/overview.md:390` and `docs/overview.md:395`. Both are now updated. The other hits were already current (`internal/vscode/keybindings.go:2`, `internal/vscode/config.go:76`) or concern reed's tmux bindings (`internal/reedengine/*`), which this change doesn't touch. `docs/research/.../grades.md` is a historical record.
- F2 (set: doc comments naming `WriteConfig`): the only comment in `keybindings_test.go` naming it was the one at line 12, now fixed. The `WriteConfig` mentions in `config_test.go` sit in that file's own tests, outside this change.

Tool calls made:
1. Grep for stale ide-behavior phrases across md/go/yaml, to rerun F1's search over the whole repo.
2. Grep for `WriteConfig` in `internal/vscode` test files, to rerun F2's search for the same kind of symbol reference.
