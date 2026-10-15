---
round: 1
focus:
  - "internal/boardengine/siblings.go and board.go: does the sibling computation run under the write lock, and is a sibling reported only when the new depends_on entry is one the sibling already waits on and the two share a module label?"
  - "internal/boardcli/cli.go: does the envelope's notice name the sibling, the shared dependency and the shared labels, and does the write still succeed with exit status unchanged?"
  - "internal/boardengine/siblings_test.go and internal/boardcli/cli_test.go: do the new tests duplicate coverage already in board_test.go, runlock_test.go or help_test.go, and is every added behavior covered?"
  - "Docs and skill: do boardengine/boardcli doc.go and plugins/ly/skills/board/SKILL.md match the built behavior, and does the test-budget.yaml change match the new top-level test count?"
---

Seed focus from the plan's two cards and the changed-file list; no prior round exists.
