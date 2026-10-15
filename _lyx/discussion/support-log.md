# Support log — seed-ide-interactive-chain

## Interview

A darn run holds no interview: the scope comes from the board entry, its operator decision of 2026-10-15, and a reading of `internal/vscode/config.go` and `internal/ideengine/spawn.go`.

## Rejected alternatives

- **Switch `SeedTask` to `TaskChainAttachOnly`.** The issue's first expected fix. Rejected: the operator decided the folderOpen chain goes out entirely and that one seeding serves every caller, so a per-caller chain choice is the duplication the decision rules out.
- **Keep a folderOpen `reed attach` task.** Rejected: runs are watched in their own terminal window, and attaching on folder open is the automatic behavior the operator removed.
- **Write no tasks.json.** Rejected: the operator welcomes an on-demand `lyx reed attach` shortcut, and a regenerated file is what overwrites the stale chain in existing worktrees.
- **Seed a keybindings.json entry that runs the attach task.** Rejected: the operator's view is not a decision, and the keybindings block is user-global and dedicated to forwarding reed's tmux keys; the Run Task entry already gives the on-demand route.

## Review rounds

_No rounds yet._

## Question ledger

- **Q:** Which chain should the driven pair get? **Resolved (operator decision):** none; no folderOpen task for any caller.
- **Q:** Does the shortcut need a preceding `reed up`? **Self-pick:** no; `reed attach` boots a cold worktree's session itself.
- **Q:** Remove reed's `--unless-name` flag, now without a lyx-generated caller? **Self-pick:** no; it is an operator-facing reed flag and out of scope.
