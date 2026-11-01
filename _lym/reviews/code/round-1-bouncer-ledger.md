---
round: 1
ledger:
  - key: root-slug-collides-with-root-project-file
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

F1 (round 1): a board project with slug `root` renders to the same file as the root project, so `Render` silently overwrites it.
The review's fix text fixes the result: refuse the reserved slug in `NewTask`, build the root file name from the `RootSubject` constant, add a refusal test row, and update the `doc.go` claim.
