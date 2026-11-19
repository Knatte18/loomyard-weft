```yaml
lens: forced-edits
outcome: reported
```

1. BLOCKING. Card 6, deletion of `internal/boardengine#isRunAgentSession`: a second caller is left uncovered and the build breaks.
   - Evidence: `internal/boardengine/recordupsert.go:29` calls `isRunAgentSession(b.strandName)` inside `internal/boardengine#Board.UpsertTaskWithRecord`.
   - Card 6 lists only `internal/boardengine#checkTaskCreation` under `**Edit:**`, so `caller-uncovered` fires and the package stops compiling.
   - Fix: add `internal/boardengine#Board.UpsertTaskWithRecord` to card 6's `**Edit:**`, or keep the unexported function as a thin wrapper.

2. BLOCKING. Card 1, `Outputs` gains `Labels` and `Config.Outputs` fills it: the existing table test that compares the whole struct breaks.
   - Evidence: `internal/boardengine_test#TestOutputs` (`internal/boardengine/config_test.go:345-380`) checks `reflect.DeepEqual(tt.cfg.Outputs(), tt.want)`, and its `want` values set no `Labels`.
   - `labelNames` returns `make([]string, len(labels))`, which is a non-nil empty slice even when no labels are configured. So `Labels` comes back as `[]string{}` while `want` holds nil, and both rows fail.
   - Card 1 does not list `TestOutputs`. It should, with each row's `want` gaining `Labels`.

3. BLOCKING. Cards 4 and 5, the new test package `internal/boardserve` has no `TestMain`.
   - Evidence: `pattern/PATTERN-test-isolation.md:7,13-17` requires every test package to have a file declaring `TestMain` that calls `gitkit.HermeticGitEnv()` and then `tmuxkit.Main`. The guards are in `cmd/lym/testmain_test.go`, `cmd/lym/hermeticenv_test.go` and `cmd/lym/tmuxisolation_test.go`.
   - Card 4 creates `internal/boardserve/server_test.go` and card 5 creates `run_test.go` and `owneronly_windows_test.go`, but no card creates `internal/boardserve/testmain_test.go`.
   - Fix: create it on card 4, with its Intent naming it as `untagged` (a support-only file).

4. MEDIUM. Card 7, the `tasks.json` and key-binding changes leave the operator overview wrong.
   - Evidence: `docs/overview.md:582` says "The generated `tasks.json` holds one on-demand `reed attach` task", and `:606-608` describes the `lym:begin` block as binding only `ctrl+alt+a`.
   - After card 7, `tasks.json` holds `board open` too, and the block gains `alt+shift+9` and `alt+shift+0`. No card lists `docs/overview.md`.
   - Fix: add it under card 7's `**Extends:**`/`**Edit:**`, and card 9 should add the workspace file's `beam start` task to the same passage (`:597-601` describes the workspace files).

5. LOW. Card 6, `internal/boardcli#Command`'s file comment and `rerender`'s help text describe the render outputs and verbs as they stand today.
   - Evidence: `internal/boardcli/cli.go:3-5` lists the verbs. `rerenderCmd`'s `Short` says "rebuild the README and design docs from board.json" (`cli.go:406`), which after cards 1 and 3 also rebuilds `board-data.js` and `board.html`.
   - No card rewords either, and card 1/3 list only `internal/boardengine` targets.

6. LOW. Card 1, Intent bullet on escaping: the text, as rendered in the reviewed file, names the escapes as "`<`, `>`, `&`".
   - D1 asks for the JSON unicode escapes, a backslash and `u` followed by `003c`, `003e` and `0026`.
   - Evidence: `_lym/plan/01-board-view-projection.md`, Intent's first bullet, "is written as `<`, `>`, `&`". The implementer could read that as no escaping at all.

Tool calls made:
- Grep across `internal`/`cmd` for render file-set asserts, `Outputs` comparisons, `isRunAgentSession`, `BuildWorkspace` callers, `reed attach` in tests and runproc role users. The pre-read held no caller list for these.
- Read `config_test.go:355-380` and ran quarry for `TestOutputs`, `recordupsert.go` and `RunProcDir`, to confirm the broken `DeepEqual` row and the glyphs, which the pre-read lacked.
- Grepped `PATTERN-test-isolation.md`, the `cmd/lym` testmain guards, the `lymdirs` doc and docs/contracts, for registration and doc forced edits the pre-read had not covered.
- Grepped the fabric doc, pattern files and `.md` files for the folder-namer seam, key bindings and Replay claims, to find docs describing what cards 3, 7 and 8 change.
- Read `docs/overview.md:576-612`, the VS Code passage the previous grep located.
- Grepped `docs/overview.md` for board render, workspace and replay passages; it found none.
