---
round: 4
focus:
  - "Every plan site that reads a run's last-transition code tip from origin's records branch (the overview's `code-tip-is-warp-sha` decision, `lastTransitionCodeTip`, `OriginCodeTip`, `ReturnToPreviousTransition`): does each select only the commits a run transition wrote, given that `Fabric.Commit` (`internal/fabricengine/recordsgit.go` `commitWeftLocked`) appends `Warp-SHA` to every weft commit, including those from `lym fabric commit`, `sync` and `push`? Check the `Commit` closures of the transition seams write whatever marker the readers select by."
  - "Cards that name a test setting an unexported hook or variable of a package from an external `_test` package: does some card create the exporting helper (the `export_test.go` convention in `internal/fabricengine`) so the named test row can be written?"
  - "Cards that remove a field or seam (`Spawn`, `guardedSpawn`): is every doc comment, wiring comment and test that describes the removed item a target of some card?"
---

The hub's plan gate checks targets and verify lines mechanically; the questions above are about what it cannot check.
