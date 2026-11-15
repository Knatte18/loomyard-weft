# Codex as darn writer, replayed on a landed task, on Neve

`codex-writer-bench-neve` · Task · research, codex, shuttle · running · Finalize

## Goal

Level 2 of `codex-support-levels` (R2): can Codex do the darn writer's work on real lym code as well as Claude did?
The bench replays only the writer, never a whole run: no other producer runs, and nothing lands or is pushed.

## Input

The darn light task `ide-beam-readme-preview`, landed as `824eb2b6`.
Its Claude writer's work is the answer key: the landed diff, and the run's records on its records branch (writer cost, review rounds and findings).

## The one run

1. Check out the parent of `824eb2b6` into a scratch worktree under this task's `.scratch/`.
2. Build the writer's prompt the way Darn-Write built it for the landed run, from the task's body as it stood then, pointed at the scratch worktree.
3. Run it once through `lym shuttle run --model 'astra[high]'` with that prompt.
4. In the scratch worktree, run `lym gate test` on the packages the diff touches.

Exactly one Codex session; no Claude run, since the landed run is the comparison.

## Read

- Did it finish the work, and does the gate pass on its tree?
- Its diff against the landed one: scope, correctness, comments and prose in the house tone.
- Cost, wall time and turns against the landed writer's.
- Any sandbox, skill-load or idle-wake fault on the way (`sandbox_mode: auto`, `codex-idle-wake-extra-reminder`).

## Bounds

Change no production code and nothing under Bolt; remove the scratch worktree at the end.
The report goes under `docs/research/codex/`; the verdict on level 2 is the operator's.
