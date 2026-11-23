---
round: 2
focus:
  - "In internal/boardengine/board.html, `projectSelect` builds the create and edit forms' project options from the view's project list, which `viewProjects` in internal/boardengine/view.go now extends with done projects; settle whether any option offered can be refused by `checkProjectField` in internal/boardengine/project.go."
  - "In internal/boardengine/view_test.go, `viewFixture` and `TestRenderBoardData`: settle whether a done project nested in another done project, and the order among done projects, are pinned so that hard-coding the root handle as a done project's parent fails a test."
---

Focus is limited to the viewer's project list and its tests.
