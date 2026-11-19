---
round: 1
focus:
  - "internal/boardcli/intake.go `intakeList`: settle whether a failed `ListIssueComments` mid-loop fails the whole list as intended, and whether the compact (non-full) path still makes no per-issue request."
  - "internal/boardcli/intake.go `carriedComments` and `inboxIssue`: settle whether the shared filter leaves the import path's behavior unchanged, and whether `list --full` comments match exactly what import carries into the entry's body (order, pointer filtering, `[]` rather than null when none)."
  - "Scope against the board entry: settle whether the diff makes the entry's two asks, import carrying comments under a heading of their own and `list --full` printing them, or whether import's carrying was already present at the review base and the diff only adds the list half; check for any change beyond those asks."
  - "contracts/orchrole/role-intake.md and plugins/ly/skills/board/SKILL.md: settle whether the new note-splitting rule and the changed upsert limits are consistent with the `list --full` and import behavior the code lands, and that the one-line-per-issue output format change is coherent with the rest of the role."
  - "internal/boardcli/intake_test.go: settle whether the changed per-issue-request assertion and the `CommentListFailure` case still check what the test's name states, and whether any assertion was loosened relative to the review base."
  - "Doc and help text (`doc.go`, the `list` command's long help and `--full` flag text, SKILL.md): settle whether each states the new `comments` shape consistently with the JSON the code prints."
---

Seeded from the committed diff and the board entry alone; no prior round exists.
