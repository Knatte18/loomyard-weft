# A beam member's run records the prime orch as its parent orch

`beam-member-parent-is-prime` · Note · bug, batten, landing, orch

Imported from [issue #232](https://github.com/Knatte18/loomyard/issues/232).

## Seen

In the DrillOpPlan hub on 2026-11-24, the run `models-stick-slip-map`, a member of the beam `models-torque-drag` and spawned from the beam's worktree, records the prime's orch `dop:orch` as its parent orch.
Its PR-Review sent the `[pr-review request]` for PR #52 (into the beam branch `hanf/models-torque-drag`) to `dop:orch`, as ask 1 from the PR-Review session and as ask 2 from `dop:LOOM`.
The beam's orch `dop:models-torque-drag:orch`, which owns and released the run, reviewed the PR, and its answer was refused:

```
caller dop:models-torque-drag:orch is neither the run's parent orch nor the strand the ask went to
```

The beam's dobby sends its notices for the same run to `dop:orch` too.
Knatte18/loomyard#231 shows the same gap in the tackler and driver prompts of a beam member.

## Expected

A beam member's run records the beam's orch as its parent orch, so its parent-review asks, notices and escalations go to the beam's orch and that orch's answer is accepted.

## Workaround

The prime orch forwards each ask and notice to the beam's orch, and runs `lym loom parent-review answer <slug> --ask <n> --satisfied` in the task worktree on the beam orch's verdict.
