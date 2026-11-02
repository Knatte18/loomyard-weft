# loom Parent-Align halts blocked when the fixer reports verdict fixed

`parent-align-fixed-reads-dirty` · Note · bug, landing

Imported from [issue #6](https://github.com/Knatte18/loomyard/issues/6).

## What happened

`lym loom step` ran Parent-Align after merging the parent branch into the task branch.
The merge was committed (`fabric: mutation kind=merge_committed`) and the post-merge landing gate passed (`landingshed: post-merge landing gate passed`, verify command exit 0).
The loom then started a fixer session for Parent-Align.
The fixer ran about 40 seconds, reached `idle-done` (`loop=done`), and wrote `.lym/loom/Parent-Align-fixer-report.md` opening with `verdict: fixed`; the report's parent-change review listed no problem.
The loom logged `landingshed: fixer run ended ... end=dirty spent=1 cap=2` and then halted the run as `blocked` with anomaly `escalation-to-human`:

`landing stop: the fixer session of Parent-Align ended without a usable turn (dirty) ... way forward: fix or clean the task branch, then lym loom resume`

## Why this is abnormal

The gate passed, the fixer finished a clean turn, and its report says `fixed`, yet the loom classified the end as `dirty` ("without a usable turn") and escalated to a human after one of two allowed fixer attempts.
The stop message does not say what was dirty (uncommitted files, an unmerged index, or a missing handoff), so the operator cannot tell whether the branch needs work.
Either the end-state classification is wrong when the fixer has nothing to change, or the message hides a real working-tree condition.

## What to do

- Check how `landingshed` derives `end=dirty` for a fixer that finishes with `verdict: fixed` and no changes, and treat a passed gate plus a finished fixer turn as a clean end.
- When the end really is dirty, name the cause (paths or state) in the halt reason instead of the generic "without a usable turn".

Provenance: task `reed-status-buttons`, note `loom-halt.md`.



_Moved from #911 in the repository before it was recreated._
