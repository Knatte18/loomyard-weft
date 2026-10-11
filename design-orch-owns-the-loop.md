# Orch loop: one land command and orch start --continue

`orch-owns-the-loop` · Note · feature, batten, orch

## Goal

An orch session runs the whole loop with lyx commands and its committed role prompt.
Detached batten starts, run notices and the committed role prompt have landed; two pieces remain.

## One land command

The steps after a landing (pull in the prime, deploy, config reconcile, intake, prune, the batten teardown check, the fabric commit) become one lyx command.
Deploy stays operator-triggered, so the command either stops before deploying or deploys only when told.

### Batten lands itself (operator, 2026-10-11)

Stronger form: batten runs the land steps itself after Finalize and wakes the orch only when a step fails, so the `done` notice and the orch's five to eight calls per landing go away.
Measured 2026-10-11: orch spend follows its call count, since every call reads the whole context (see `orch-token-spend`).

### What wakes the orch (proposal, 2026-10-11)

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

## Continue the last orch session

After a reboot the operator wants the same conversation back, as `claude --continue` gives, not a fresh session reading a handoff.
Today that takes `lyx orch start --adopt <session-id>`, with the id dug out of `~/.claude/projects/`.
`lyx orch` records its session id and offers `lyx orch start --continue` (or makes it the default for a dead strand) to adopt the last orch session.

## Done when

- The orch lands a run with one lyx command.
- `lyx orch start --continue` after a reboot resumes the last orch conversation.
