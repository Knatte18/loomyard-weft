---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: "`ViewProject` carries `Done` (JSON `done`), `viewProjects` sets it for the done-project loop only, and `projectSelect` in board.html skips a project whose `done` is true, so the dropdown no longer offers a done project."
  - id: F2
    status: resolved
    answer: "`viewFixture` adds done project P5 (epsilon) inside done project P3 (gamma), and the project-list subtest expects P5 with `Parent: \"P3\"` after P3, so hard-coding the root handle fails it."
  - id: F3
    status: resolved
    answer: "Settled by the added entry \"The task fixes the board viewer; the list --text listing stays as it is\"."
---

No new findings this round.
The diff makes the change the amended entry asks for:
a done project's members, done ones included, now stand under their own project in the viewer and no longer fall back to the root.
`lym gate test ./internal/boardengine` passes with 867 passed, 0 failed and 7 skipped.
The mutation report kills every mutant.

## Focus directives

- **`projectSelect` against `checkProjectField`.**
  `projectSelect` offers `(unchanged)` on edit, `(no project)` and every project in `state.view.projects.slice(1)` that is not `done`.
  `slice(1)` drops the root, and `walk` adds only open projects (`childProjects` filters `!isDone`), so no option names a done project.
  The `isDone` refusal in `checkProjectField` therefore cannot fire from the dropdown.
  Its other refusals are no-entry, non-project, self, `RunHolds` and a project cycle.
  The first two cannot fire because every option is a project slug from the board.
  The last three can fire only through choices the page offered before this diff:
  editing a project to stand in itself or a descendant, or choosing an open project a run holds.
  The diff does not change them, so they are no finding on a changed hunk in this incremental round.
- **Nested done project and order in `viewFixture` and `TestRenderBoardData`.**
  P5 (`Project: "gamma"`, done) is expected with `Parent: "P3"`, so returning `rootProject.Handle` for a done project's parent fails the subtest.
  `reflect.DeepEqual` over the whole slice pins the order: open projects P1, P2 and P4 come first, then P3 before P5 in README order.
  It also pins `Done` on every row, so dropping the field, or setting it on an open project, fails too.

## Rejected

- Open projects held by a run, and a project's own descendants, still appear in the edit dropdown and are refused by `checkProjectField`: the page offered them before this diff, and the diff does not change that, so they are no new BLOCKING finding on a changed hunk.
- The assistant reviewer's end marker reads `timeout`, so its report was not read.
