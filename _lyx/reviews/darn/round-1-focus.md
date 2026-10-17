---
round: 1
focus:
  - "internal/tendercli/gate.go: does the `preflight.BoardLyxPresent` check run before any write at the resolved hub, and does it leave a real hub's gate installation unchanged, including standalone mode and plain-repository callers?"
  - "internal/tendercli/gate.go: does the new `preflight` dependency fit the package's import direction under PATTERN.md, and is `BoardLyxPresent` the same predicate hub reconcile uses, as the summary claims?"
  - "internal/tendercli/cli_test.go: does the extended test cover the issue's case (a linked worktree under `<prime>/.claude/worktrees/<name>`), and does it duplicate coverage of an existing test?"
  - "internal/tendercli/doc.go and _lyx/landing/summary.md: does the doc line match the new behavior, and does each claim in the summary, such as 'the only change in behavior is that no marker is written there', hold against the diff?"
---

Small three-file diff; scope is the board entry for issue #697.
