# lym fabric push lock held past 30s blocks the loom run at PR-Review as escalation-to-human

`push-lock-busy-blocks` · Task · bug, fabric, landing · running · Code-Burler-Review

Imported from [issue #37](https://github.com/Knatte18/loomyard/issues/37).

What happened: the PR-Review row halted the loom run as blocked with 'pushing the task branch failed: fabricengine: push lock busy: <weft>/.weft/fabric.push.lock still held after 30s; neither side was pushed'. The way forward named was to run lym fabric push in the task worktree, then lym loom resume. The run had been through 17 history entries and had just finished Describe.

Why it is abnormal: the push lock sits in a weft directory shared across worktrees, so another session's push or a stale lock holder can stall a push; a 30s wait then ends the whole run as an escalation to a human, though contention is transient and a push is idempotent. Nothing in the note says which process held the lock, or whether it was stale.

What to do: retry the push with backoff for longer, or re-step the row automatically before blocking; when the lock is still busy, report the holder (pid, age) and clear a lock whose holder is dead.

Provenance: task pinned-solvers-everywhere; notes loom-halt-2.md