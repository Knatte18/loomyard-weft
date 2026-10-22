---
round: 1
focus:
  - "Card 3 and its readers: does every strand-liveness reader in reed take `engineAliveIDs` while pane-level checks (Selvage health, pane options, title repair) keep `aliveIDSet`, as the plan's Shared Decisions state?"
  - "Card 8: does every writer and remover of `cycle-request` compare and remove under the orch state lock (`Paths.StateLockPath`), with no path left that touches the marker unlocked?"
  - "Card 13: does `stop` take `start.lock`, and does status restart a dead watcher under it without a double start?"
  - "Per-card mechanical checks: for each card's groups, did the type-specific check run and pass against that group's own targets, including `assert-no-callers` on any `Delete` group and no left-over old name on any `Rename` group?"
  - "Test economy: do the new tests across cards 1 to 13 duplicate an existing test, or leave an added behavior (for example `lyx orch withdraw`, `held_by_pause`, `strand_elsewhere`) uncovered?"
---

Seeded from the plan's Shared Decisions and Card Index.
Cross-card invariants come first because a single-card read misses them.
