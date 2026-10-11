# Batten lands itself; the orch wakes only for judgement

`orch-wake-policy` · Task · feature, batten, orch

Split out of note `orch-owns-the-loop` (2026-10-11); its session-continue part is now note `orch-continue`.

## Seen (2026-10-11)

Orch spend follows its call count, since every call reads the whole context (see `alignment-agent`).
After each landing the orch runs pull, deploy, config reconcile, intake, prune, the batten teardown check and the fabric commit by hand, five to eight calls, and after `lyx loom approve` it runs a `lyx loom start` retry loop.
Many notices name their own action and need no judgement.

## Idea: batten lands itself (operator, 2026-10-11)

Batten runs the land steps itself after Finalize and wakes the orch only when a step fails, so the `done` notice and the orch's calls per landing go away.
Deploy (`update-plugins.sh`) is part of it: today only the hub orch runs it after a landing (CLAUDE.md), so the rule moves with the step.

## What wakes the orch (operator agreed, 2026-10-11)

The orch is woken only for what needs judgement or its context:
- parent-review requests;
- review escalations a perch's auto-accept does not settle, and design calls;
- PR-Gate, for the check against the board goal and the operator's decisions;
- a halt whose cause or way forward is unknown.

Handled without the orch:
- a finished run: batten lands it;
- after `lyx loom approve`: batten resumes the child itself;
- a halt whose error names its action: a Publish verify failure gets one automatic goto Webster-Burler and resume, and a second failure wakes the orch; a GitHub "cannot lock ref" is retried;
- the per-worktree `loom.yaml` landing warning (gone with `task-run-settings`);
- starting ready tasks (`orch-split`).
