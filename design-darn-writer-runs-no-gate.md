# Darn-Write writer runs no gate itself when the cold fixer is on

`darn-writer-runs-no-gate` · Task · bug, darn, stencil, gate · running · Record-Write

The Darn-Write writer still runs `lym gate test` and fixes its failures in its own session, at full context, though the cold gate fixer was meant to take that work off it.

## What is wrong

The cold gate fixer (`internal/gatefixer`, landed in `a9536680`) clears a failed Darn-Write `verify` gate after the writer hands off, and the writer hears only what the fixer leaves.
That commit switched the Plan-Write and Discussion-Write prompts to a "gated" self-check fragment that runs no validator when the row's gate has a cold fixer (`LoomFragmentSelfCheckPlanGated`, `LoomFragmentSelfCheckDiscussionGated` in `contracts/stencils/stencils.go`).
Darn-Write's prompt was left out: `contracts/stencils/darn/darn-template-write.md` Step 4 still tells the writer to run `lym gate test` after each commit, rerun it, and leave each commit green, and tells its subagents to run the gate on their packages.
Seen in task `webster-subpath-anchor`: the writer ran the gate and fixed failures itself before and after the fixer.

## Expected

- When `darn.gate_fixer` is set and the worktree has a pair slug, the Darn-Write prompt has the writer and its subagents run no `lym gate test`; the row's `verify` gate and its fixer take the failures, and the writer acts only on what the fixer leaves.
- With the fixer off, or with no pair slug, the prompt text stays as it is today.
- Follow the Plan-Write pattern: a marker in `darn-template-write.md` filled by the darn engine with one of two fragments, chosen on the same condition the row gate uses to attach a fixer.
- A test pins both renderings.
