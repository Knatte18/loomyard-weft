---
round: 1
focus:
  - "internal/fabricengine/remove.go: the second look after sibling teardown deletes a refilled directory. Settle whether it can delete a directory that is a live registered worktree or holds user work, and whether a failed delete reaches StrayPath on every path."
  - "internal/fabricengine/prune.go: Unowned reports a hub directory named as a pair slug that no registration covers. Settle how a slug is recognised, whether an unrelated hub directory can be reported, and that nothing is removed."
  - "internal/battencli/wire.go and cancel.go: Worktree-Create halts on a stray task directory and cancel --remove warns of one. Settle that the halt names the path and the way forward, that a registered worktree never trips it, and that the warning fires only when the removal left a directory."
  - "internal/boardcli/cli.go and fabricengine PairWorktreeStands: board rename sees a pair only where a registered worktree stands. Settle that a real pair, including a branch-only or adopt-only one, still refuses a rename."
  - "Scope against the board entry's four Expected bullets: check each is made by the diff, and that the diff makes no change the entry does not license. The entry allows Worktree-Create to refuse or to clear ignored build output; check the chosen way is consistent."
  - "Test coverage of each behavior the diff adds, and the two tests updated in 37aaa304: settle that each updated test still checks the behavior it was written for."
---

Seed focus for round 1.
The answer key is the decision record's `## Goal` alone; its other sections are empty.
