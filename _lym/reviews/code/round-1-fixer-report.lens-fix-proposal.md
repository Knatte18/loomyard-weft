```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1:** The restored "Mail from `<shortname>:SLACK`" form in `contracts/orchrole/role-slack.md` agrees with the intro line. I found no case where it breaks the file, and it blocks a `[slack]` body from any other sender.
- **F2:** The new wording in `docs/overview.md` is accurate.
  - The role names `lym help index`.
  - The `ly:cli` skill carries the index.
- **F3:** The marker check holds in every rendered role.
  - `contracts/orchrole/orch-role.md:154` carries `{{.decision_note_title}}`.
  - `internal/orchengine/prompt.go:78` fills it with `Decision for <slug>:` on every `RenderRoleFile` call.
  - The start, resume, watcher and compact tests all read a role written by that path, so none can fail on a missing string.
  - The `len(role) == 0 ||` clause in `internal/orchcli/start_test.go` is redundant with the `Contains` check. It does no harm.

Tool calls:
1. `grep "Decision for"` in `contracts/orchrole/` and `internal/orchengine/prompt.go`. My context did not show where the string comes from.
2. `grep DecisionNoteTitle|decision_note_title|<slug>:` in the same paths. I needed to confirm the role carries the marker that the render fills.
