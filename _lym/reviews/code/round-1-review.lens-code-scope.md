```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. For each change I list the decision it traces to, then the decisions no change builds.

**Change → decision**
1. `internal/claudeimports` (new package), `PATTERN-leaf-packages.md` entry: D8 and D9, card 1.
2. `agents-prompt` target in `target.go`; `passagesOf`, `promptTargetsFor` and the `Show` refusal in `render.go`; `heddlecli` help; `overview.go` intro; `heddleengine/doc.go`: D7, card 2.
3. `fabricengine.NotifyWorktreeMadeAt`, the `walk.go` call, `hubreconcile/doc.go` and the `PATTERN-heddle-instructions.md` line: D7, card 3.
   - The `worktreeseam_integration_test.go` edits and the `fabricengine/doc.go` edit are forced by this change.
4. `codex-file` carrying both files with imports expanded, `importrecord.go`, `RenderRequest.Root`, the `seam.go` `TookIn` matcher, `Root` in `heddlecli.renderInto`: D9, card 4.
5. `developerinstructions.go`, the `codexengine.go` assembly, `command.go` `developer_instructions`, and the prompt file holding the prompt alone: D5, card 5.
   - The `docs/overview.md` wording and the `guards.go` header comment are forced by this change.
6. `subagents.go`, `agents.*` settings, `max_subagent_threads`, `template.yaml` and `config.go` validation: D6, card 6.
   - `plugin_hook_timeout` and its validation belong to D8.
7. `shuttleengine/usersetup.go` (seam types) and `claudeengine/usersetup.go` (global file, files above the root, skill listing, config-dir degradation): D8 plus the "drop only the parts that need the config dir" entry, card 7.
8. `pluginhook.go` and its unix and windows files, with `CLAUDE_PROJECT_DIR` set to `req.WorkDir`: D8 plus the two `CLAUDE_PROJECT_DIR` entries, card 8.
   - The windows file is forced by the unix one.
9. `codexengine.Seams`, `usersetup.go`, `NewFromConfig` and the `shuttleengines.go` wiring, `PATTERN-shuttle-provider-seam.md`: D8 and the Exports line, card 9.
10. `instructionfiles.go`, `project_doc_max_bytes`, the pointers: D9, card 10.
    - The `unreadClaudeFiles` rule that also names a CLAUDE.md on the root-to-workdir path that imports files has no source in D9. It is listed as finding S1.
11. `heddlecli/verbs.go` `set` help rewording: none. It is listed as finding S2.
12. The `heddleengine/render_integration_test.go` one-line edit is forced by the `Root` field (change 4).
13. Test files for each change are forced by that change.

**Decision → changes**
- D5: change 5.
- D6: change 6.
- D7: changes 2 and 3. The shipped-entry move is already in the tree before the range, at `22b74f24`, and `shipped.json` holds both entries on `agents-file`. The diff correctly changes nothing there.
- D8: changes 1, 7, 8, 9.
- D9: changes 1, 4, 10.
- Added entries, config-dir handling, `CLAUDE_PROJECT_DIR`, and directory-based unreadable rows: changes 7 and 8, and the rows in `claudeimports_test.go` and `usersetup_test.go`.
- Added entry on the assist-trial schedule risk: no diff expected.
- Added entry on round-3 findings answered by D1, D4 and D12: those belong to sibling parts, so no diff is expected here.
- No decision lacks a building change.

**Findings**

- **S1 — LOW, scope.** `internal/shuttleengine/codexengine/instructionfiles.go:597-601` (`claudeFileIsUnread`).
  - D9 names two unread categories: a CLAUDE.md below the workdir and one beside an `AGENTS.md`.
  - The code adds a third: a CLAUDE.md on the root-to-workdir path with no `AGENTS.md`, whose imports resolve to files.
  - It also adds a heddle-marker exception for the workdir (lines 595 and 616-620) and the `TestPrepare_InstructionFiles` rows `ClaudeFileWithAnImportIsNamed` and `LoneClaudeFileWithNoImportIsNotNamed`.
  - Neither the record nor card 10's decisions licence this, as far as my context shows. The marker exception may be implied by D9's "beside an AGENTS.md" with the `codex-file` override carrying both files.
  - Fix: either remove the import-based rule, or ask the parent to record it with `lym loom decision add`.
- **S2 — NIT, scope.** `internal/heddlecli/verbs.go:223-235`, the `set` help text.
  - Besides adding `agents-prompt`, the diff drops "the prime, beams and task pairs" and the clause about `render_warnings` for "a failure to list the worktrees".
  - No decision asks for that rewording. Card 2 may cover part of it; my context does not show it.
  - Fix: restore the dropped wording unless the plan's card 2 carries it.

Tool calls: none; the manifest context sufficed.
