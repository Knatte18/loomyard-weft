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


## From issue #441

Imported from [issue #441](https://github.com/Knatte18/loomyard/issues/441).

**What happened**
After `lyx webster record-batch` failed on a config-load error (see the batcher.yaml stale-key issue), `lyx webster begin-batch 1` was refused with: `batch ... already has a report at _lyx/webster/reports/<batch>.yaml and state.json records the batch as begun and not terminal — begin-batch never overwrites finished work; a report left behind by a crashed session is consumed by lyx webster record-batch 1 (or lyx webster recover-batch 1 for a recovery batch), and a stuck batch escalates via lyx webster recover-batch 1 (which archives the report)`.

**Why it is abnormal**
A record-batch refusal that is not about the report itself (here a config-load failure) leaves the batch begun with a report on disk, so the natural retry path through begin-batch is blocked.
The refusal message lists several remedies without saying which applies after a record-batch config failure, and the Webster session ended up in a dead end that needed the operator side to intervene.

**What to do**
- Make record-batch fail before touching batch state when its config does not load, or make the state recoverable by re-running record-batch once the config is fixed.
- Have the begin-batch refusal say which remedy applies given the recorded state (for example, "re-run record-batch after fixing config" versus "recover-batch").

Provenance: task slug `lifecycle-waits`; notes `webster-refusal-begin-batch.md`, `loom-halt.md`.
