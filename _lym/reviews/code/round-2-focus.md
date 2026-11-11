---
round: 2
focus:
  - "plugins/ly/skills/board/SKILL.md, the promote section: does it teach agents that a Sketch's dependencies go in promote's depends_on, since promotion releases the task before a later set-deps could land?"
  - "internal/boardcli/cli.go and internal/boardengine PromoteTo: does a test assert that promote's success envelope carries a notice for a sibling task sharing the promoted note's depends_on, and that the engine returns that sibling?"
  - "internal/boardengine/store.go promoteTo, the already-a-task branch, and internal/boardcli promote payload parsing: are an equal depends_on, an empty non-nil depends_on on a task with dependencies, a null depends_on and a non-string element each pinned by a test row?"
---

The entries ask where to look; they set no severity and no verdict.
