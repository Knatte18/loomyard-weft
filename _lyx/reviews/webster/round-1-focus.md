---
round: 1
focus:
  - "internal/boardengine/runlock.go and Board.boardCriticalSection in board.go: settle whether the snapshot clones every slice field, the check runs after fn and before Save, noWrite skips it, and a refused write leaves board.json untouched."
  - "checkRunLocks: settle whether every field the card lists (Title, Kind, Labels, Issues, Recipe, DependsOn, Isolated, Brief, Body, ShortName) is compared, nil equals empty for slices, an absent entry is refused, and the prune exception drops only slugs removed as done."
  - "RunLockedError.Error: settle whether the message names slug, status, the note way forward and the lyx batten status check from the prime worktree, using only board and batten verbs (PATTERN-no-denied-recovery)."
  - "internal/battencli/wire.go: settle whether the claim is the CreateWorktree closure's first statement, composed through boardengine.RunStatus, warns and continues on failure, and lives only in battencli (PATTERN-shed-producer-seam, PATTERN-told-geometry)."
  - "boardengine.OpenHub and Board.SetRunStatus, with the loomcli call sites: settle whether openHubBoard is fully removed, the hub-containment and told-geometry rules hold, and SetRunStatus skips absent and done entries under the write lock."
  - "Test economy across runlock_test.go, board_test.go, wire_seams_test.go and the intake row: settle whether each new test covers behavior no existing test covers, whether every added behavior has coverage, and whether the test-budget.yaml rows match the new top-level tests."
  - "Docs and comments: settle whether the boardengine and boardcli doc.go, battencli cli.go and the board SKILL.md changes match the code, and whether new doc comments follow the repo's Go comment conventions."
---

Seeded from the plan's three cards and the committed diff.
