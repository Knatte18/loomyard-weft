---
round: 4
focus:
  - "In internal/testkit/tmuxkit/tmuxkit.go, settle whether `sweep` (called from `afterRun`) guarantees that the pane process trees of every swept tmux server are gone before the leftover scan runs, or relies on the SIGHUP cascade landing within `leftoverGrace`; check that a test pins a pane process surviving `kill-server` alone (for example a `trap '' HUP` sleep)."
  - "In internal/lyxcwd/lyxcwd.go `gitWorktreeRoot` and internal/dotgit/dotgit.go `FindRoot`, settle whether a worktree root reached through a symlinked prefix resolves to the same real path the retired `git rev-parse --show-toplevel` returned, so `HubPath` and the reed socket key agree for both paths, and whether a test row starts from a symlink."
---

Both entries restate the Publish-verify and parity concerns at their sites so the reviewer judges the current code fresh.
