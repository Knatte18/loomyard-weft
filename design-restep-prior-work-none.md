# Re-step prompt after an interrupted run reports prior work as (none) though the branch carries the earlier spawn's commits

`restep-prior-work-none` · Task · bug, loom, stencil · done

Imported from [issue #278](https://github.com/Knatte18/loomyard/issues/278).

A run was interrupted by a machine reboot at the Darn-Write row and re-stepped.
The re-spawned agent's prompt said the prior work was `(none)` and that the branch held no work of this session's.
The branch already carried three commits from the interrupted spawn, and `.scratch` held a half-finished measurement.
The agent had to read the halt note and file timestamps to tell the earlier spawn's work apart from someone else's.

This is abnormal because the prompt's prior-work statement contradicts the branch state, and an agent trusting it could redo or overwrite finished work.

What to do: when a row is re-stepped after an interrupted-stop halt, compute the prior-work section from the commits the branch holds beyond the base and from the surviving scratch files, or state that the earlier spawn was interrupted and its work may be present.

Provenance: task count-branching-solver; note Darn-Write.md.