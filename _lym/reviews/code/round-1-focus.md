---
round: 1
focus:
  - "internal/heddlecli/verbs.go `renderEveryWorktree`: does the worktree name taken from `filepath.Base` of git's listed path always resolve through `lymcwd.NamedWorktree` for the prime, beams and task pairs, and is any live worktree of the hub skipped or mis-named?"
  - "internal/heddlecli/verbs.go `finishWrite` and `renderEveryWorktree`: the render runs after `saveLocked` releases the Bolt write lock and before the detached sync starts; does any path make a render wait on the lock, or leave a saved change unreported when the fan-out fails?"
  - "internal/heddlecli/verbs.go `renderEveryWorktree`: is a worktree with `Unwritten` files reported in `failed` with a usable cause when `report.Warnings` is empty, and are `rendered`, `failed` and `render_warnings` printed consistently on success and on partial failure?"
  - "internal/heddlecli/cli.go `ChangeStore` and `changeStore`: after `syncWarning` was removed, do the exported `ChangeStore` callers still get the sync warning, and does the new `syncAfterSave` keep the old `write` behavior (no sync on a refused change)?"
  - "internal/heddlecli/verbs.go `setOverride` and `removeCmd`: does `--no-render` reach every write path, including `--layer lym` set and remove?"
  - "internal/heddlecli/verbs.go render verb: does `renderInto` keep the behavior of the inline code it replaced, including the error text that `render` checks for `lym heddle render`?"
  - "internal/heddlecli/cli_integration_test.go: which behaviors the diff adds (skipped prunable worktree, failed worktree, `--no-render`, override layer, list failure) have no test, and which new assertions duplicate an existing test in the file?"
  - "Help text in `setCmd`, `removeCmd`, `cli.go` and doc.go, and the PATTERN-heddle-instructions.md line: do they match what the code does, and does the commit message's claim that the sync starts after the renders hold?"
---

Scope is the committed diff over `b2141fcb`.
The decision record holds only the board entry under `## Goal`, with no amendments, so the entry is the measuring stick.
The entry asks for a render of every file target into every live code worktree of the hub (prime, beams, task pairs), a report of rendered and failed worktrees, and an optional skip flag.
