# Recovery prompt gives no guidance on uncommitted changes left by the prior attempt

`webster-recovery-dirty-tree` · Note · bug, webster

Imported from [issue #406](https://github.com/Knatte18/loomyard/issues/406).

A recovery run for a Webster task segment was started with the reason `none` and an instruction to continue from the committed tree.
The worktree nevertheless held a large uncommitted body of work from the earlier attempt (dozens of files, thousands of changed lines).
The recovery prompt did not mention these changes or say whether to trust and keep them or to discard them, so the agent had to judge on its own whether the earlier work was sound.

This is abnormal because the recovery prompt contradicts the worktree state: "continue from the committed tree" is not true when the tree is dirty.
The agent kept the work only after verifying it independently (build, both test tiers, a fresh rerun of the segment's check).

Suggested fix: when lyx starts a recovery, have it detect uncommitted changes in the worktree and name them in the recovery prompt, together with an explicit keep-or-discard instruction.

Provenance: task `test-suite-prune-core`; note `10-websterengine-prune-recovery.md`.
