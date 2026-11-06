```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each changed test and the Slack bullets through concrete inputs.

- **Role-file assertions, all five call sites:** `RenderRoleFile` fills `{{.decision_note_title}}` with `Decision for <slug>:`. That marker sits in `contracts/orchrole/orch-role.md:154`, so the rendered role contains the string. If the render failed or left the marker unfilled, `os.ReadFile` or `Contains` would fail and each check would error. The five sites are `watcher_compact_test.go:521`, `resumecontext_test.go:135`, `watcher_test.go:1632` and `:1792`, and `start_test.go:456`.
- **`prompt_test.go`:**
  - The `testRoleHeading` const is gone, and grep finds no other use of it.
  - In `TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt`, the `role` variable and its `ReadFile` are removed together, so nothing is left unused.
  - `orchrole`, `os` and `strings` are still used, and the fixer's `go vet` of `orchengine` and `orchcli` passed.
  - The on-demand test still fails if a file is missing or lacks a `# ` heading.
- **`start_test.go`:** `role` is a `[]byte`, so `len(role) == 0` compiles. The `Contains` check already covers the empty case, so the length check is harmless.
- **`role-slack.md`:** all three bullets now carry the `<shortname>:SLACK` sender condition, and the file has no markers, so the render is unaffected.
- **`docs/overview.md`:** this is prose only and has no code effect.

Tool calls made:
1. `grep` for `Decision for <slug>:` and `testRoleHeading` across `contracts` and `internal`. I used it to confirm the marker string and that no stale const use is left. My context had the edits but not the repo-wide state.
2. `grep` for `decision_note_title`, `DecisionNoteTitle` and `Decision for` in `contracts/orchrole` and `prompt.go`. I used it to confirm the role template carries the marker that renders to the asserted string.
