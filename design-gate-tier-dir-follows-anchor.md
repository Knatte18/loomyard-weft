# Gate tier runs dotnet test in the worktree root instead of the anchor subdirectory

`gate-tier-dir-follows-anchor` · Note · enhancement, gate

Imported from [issue #888](https://github.com/Knatte18/loomyard/issues/888).

## What happened

In a task whose project anchor is a subdirectory of the worktree (`dotnet/`), `lym gate test --wait` ran the tier0 command `dotnet test LymCsharp.sln ...` in the worktree root.
It failed at once with `MSB1009: Project file does not exist`.
The hub's gate tiers set no `dir`, and the default run directory does not follow the anchor.

The producer agent worked around it by passing `-C <worktree>/dotnet` with an explicit target, after which all tests passed.
The loom's diff-scoped verify very likely runs the same way, since the producer hit the same failure at the handoff gate: the run halted `blocked` with `gate did not pass after 3 attempts` (anomaly `escalation-to-human`).

## Why it is abnormal

The failure is a gate misconfiguration, not a code defect, so no amount of agent retrying can clear it.
The halt's way forward, `lym loom resume`, gives a fresh verify budget but leaves the gate config unchanged.
The resumed spawn therefore hit the identical failure and halted a second time.

## What to do

- Make a gate tier's default run directory follow the task's anchor directory when the anchor is a subdirectory of the worktree.
- Failing that, detect a gate that fails identically with a missing-project error and surface it as a configuration problem, rather than as a verify failure that offers `lym loom resume` as the way forward.

Provenance: task `regression-harness`; notes Darn-Write.md, Darn-Write-2.md, loom-halt.md, loom-halt-2.md.
