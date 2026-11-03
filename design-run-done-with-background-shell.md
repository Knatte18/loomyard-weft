# Run ends with a background shell outstanding: its commit and gate result are lost

`run-done-with-background-shell` · Task · bug, shuttle, webster, gate · running · PR-Review

Imported from [issue #7](https://github.com/Knatte18/loomyard/issues/7).

**What happened**

The Master run finished with outcome `done` while a background shell was still running (outstanding for 39s).
The shell chained a `git add`, a `git commit`, `git log` and `lym gate test --wait --gate <gate>`.
The run never read the shell's result, and lym did not stop the shell.
The loom wrote a friction note ("Master's run ended with a background shell outstanding") that says shuttle removes Master's strand when the run finishes, which ends the shell only where it sits in the pane's process tree.

**Why it is abnormal**

A run reported as `done` can have an unfinished commit and an unread gate verdict.
The commit may or may not have landed, and the gate result was never checked, so `done` does not guarantee the work was committed or gated.
Whether the shell is killed depends on process-tree placement, so the outcome is nondeterministic.
The `lym gate test --wait` inside the shell may also be cut off mid-run, which can leave a gate slot held or a gate run without a reader.

**What to do**

- Make the run refuse `done` (or downgrade it) while the payload signal reports an outstanding background shell, and tell the agent to wait for or read the shell first.
- Alternatively, have shuttle stop outstanding background shells explicitly on run end and record the result, rather than relying on pane teardown.
- Check that a gate run abandoned this way releases its slot.

Provenance: task `gate-run-keeps-cache-warm`; notes: `webster-background-shell.md`.



_Moved from #912 in the repository before it was recreated._
