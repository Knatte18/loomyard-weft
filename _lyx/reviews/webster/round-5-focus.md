---
round: 5
focus:
  - "internal/testkit/tmuxkit/tmuxkit.go `sweep` and `afterRun`: after `kill-server` on a kit socket, are the pane process trees of that server guaranteed gone, including a pane whose process ignores SIGHUP (for example `trap '' HUP; sleep 300`), and does a test in the tmuxkit integration tests start such a pane, run `sweep`, and assert no leftover finding?"
  - "internal/lyxcwd/lyxcwd.go `gitWorktreeRoot` and internal/dotgit/dotgit.go `FindRoot`: for a start that is a symlink to a subdirectory of a checkout, or has a symlinked ancestor below the root, does the root returned equal what `git rev-parse --show-toplevel` gives from that directory, and does a test compare the two directly?"
---

Both entries restate the two design gaps still open at the end of round 4; the questions are about the current code at HEAD.
