```yaml
lens: code-pattern-fit
outcome: reported
```

Task: code-pattern-fit lens. I checked the repository rules against the diff.

**Rules checked**
- `CLAUDE.md` "Docs land in the same commit": **broken**, see F1.
- `PATTERN-comment-line-breaks`: held. The new and changed comments in `config.go`, `keybindings.go`, `spawn.go` and the test are one sentence per line, or split at a `;`/`:`.
- `PATTERN-test-economy`: held. `TestKeybindingsBlock` has no duplicate. `TestMergeKeybindings` and `TestSeedKeybindings` build their expectations from `keybindingsBlock()`, so none of them pins the entry texts. The "older block replaced in place" criterion is pinned by the existing `an existing block is replaced in place, markers included` row.
- `PATTERN-lyxdirs-single-declarer`, `PATTERN-lyx-temp-root`, `PATTERN-cli-cobra`, `PATTERN-stencil-ownership`: not touched.
- Declared PATTERN changes: none declared, and none were needed. The diff adds no new cross-cutting invariant.
- Commit message and departures: held. No `Pattern-Departure:` trailer is present or needed.
- `code-quality` self-contained comments: broken, see F2.
- `golang-comments` and `golang-testing` mechanics: held. The test is parallel, uses `package vscode` like its siblings, and has a `Test…` name.

**Findings**

F1 — MEDIUM, `consistency`
- Location: `docs/overview.md:390` and `docs/overview.md:395` (the `ide` module entry).
- Summary: the module table was not updated in the same commit.
  - Line 390 says the `reed attach` task is "reached through "Tasks: Run Task"". The task now also runs on `ctrl+alt+a`.
  - Line 395 describes the seeded block as "forwarding the four Alt keys of reed's tmux bindings to the integrated terminal". The block now also holds the `ctrl+alt+a` task-run entry.
- `CLAUDE.md` requires docs to land with a change in observable behavior.
- The focus asked whether any other doc still says the block is terminal-only or that the task runs only from Tasks: Run Task. This is it.
- Evidence: `docs/overview.md:390,395`. `git diff --stat` shows no `docs/` change.
- Fix: edit the `ide` entry in `docs/overview.md`. Say the task is reached through "Tasks: Run Task" or the seeded `ctrl+alt+a` binding. Say the block also seeds `ctrl+alt+a` to run the task, besides the four Alt-arrow terminal forwards.

F2 — NIT, `consistency`
- Location: `internal/vscode/keybindings_test.go:12`.
- Summary: the `TestKeybindingsBlock` doc comment refers to another symbol, "the task `WriteConfig` labels "reed attach"". The `code-quality` self-contained rule says a comment must read correctly with no knowledge of any other named symbol.
- Fix: reword to "…and ctrl+alt+a running the reed attach task", with no `WriteConfig` mention.

No tool calls were made.
