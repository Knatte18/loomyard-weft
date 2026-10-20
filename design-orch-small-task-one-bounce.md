# The orch role gives a small task one review bounce, not two

`orch-small-task-one-bounce` · Note · enhancement, orch, stencil

## Seen (orch, 2026-10-20)

The orch role (`contracts/stencils/orch/orch-template-role.md`, The board) tells the orch to give a small task `review_max_bounces: 2` and `review_hard_max_bounces: 3`, "never 1, since a second round reviews the first round's fixes".
The rule also names only the `loom` key, though a darn task takes its budget under `darn`.

## Operator view (2026-10-20)

The rationale no longer holds: a perch fixer that forks itself per lens of a twill fan (`darn_fixer_fan: fix-code`, and loom's fixer fan) checks its own fixes, so one round is enough for a small task.
The operator set the two small darn tasks `watchdog-reap-test-xdr` and `lyxbin-timeout-test-xdr` to `review_max_bounces: 1`.

## Idea

The role's small-task rule sets one bounce, drops the "never 1" reason, and names the run_settings key for the task's recipe (`loom` or `darn`).
