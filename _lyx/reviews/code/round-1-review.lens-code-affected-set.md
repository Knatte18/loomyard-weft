```yaml
lens: code-affected-set
outcome: reported
```

Task: list the affected set of the diff's changes (key-binding block, label constant, docs), then report findings.

**Affected set, per change**

- **Block gains `ctrl+alt+a` and entries carry command, args and when.**
  - Callers of `keybindingsBlock`: `mergeKeybindings`, `seedKeybindings`, and the merge and seed tests. All of them build their expected text from `keybindingsBlock`, so they are covered.
  - `internal/idecli/cli_test.go:68` asserts `// lyx:begin` and `sendSequence`. Both are still present, so it is covered.
  - The `internal/ideengine` seed and integration tests only check that the file exists, so they are covered.
  - `docs/overview.md:395` describes the block as "forwarding the four Alt keys of reed's tmux bindings to the integrated terminal". The block now also holds the `ctrl+alt+a` task binding. **NOT covered, finding F1.**
- **`attachTaskLabel` constant.** Its only users are `writeAttachTask` and the block entry, and both are covered. `TestWriteConfig*` in `internal/vscode/config_test.go` asserts the task label by its literal, which still matches. Covered.
- **`Spawn` doc.** `internal/ideengine/spawn.go:38` is updated. The doc comments on `SeedTask` and `seedKeybindings` do not say terminal-only. Covered.
- **`WriteConfig` and `writeAttachTask` docs.** Updated at `config.go:21` and `config.go:76`. Covered.
- **`docs/overview.md:390`** says the task is "reached through 'Tasks: Run Task'". With the new binding this is incomplete. **NOT covered, finding F1.**
- **Stencils, specs and fixtures.** The grep found none that mention the block or the task's invocation. Covered.

**Findings**

- **F1 — MEDIUM, consistency.** `docs/overview.md:390` and `docs/overview.md:395`.
  - Line 390 says the `reed attach` task is "reached through 'Tasks: Run Task'". Line 395 says the keybindings block forwards "the four Alt keys" to the terminal.
  - Both are now wrong: the seeded block also binds `ctrl+alt+a` to run the task.
  - `CLAUDE.md` requires `docs/overview.md` to change in the same commit when observable behavior changes, and the decision record's scope calls for the docs to stop calling the block terminal-only. Update both sentences to name the `ctrl+alt+a` binding.
  - Evidence: the diff touches only `config.go`, `keybindings.go`, `keybindings_test.go` and `spawn.go`, and `docs/overview.md` lines 390 and 395 are unchanged.

**Tool calls**

1. `grep` across the repo for "Run Task", "four Alt", "Alt keys", "terminal key" and `alt+up` outside the changed files. My context did not hold the other docs, stencils or fixtures that might repeat the old wording.
