# Beam reed session gets the task button group instead of beam

`beam-reed-task-buttons` · Task · bug, reed, pawl · running · Finalize

Imported from [issue #85](https://github.com/Knatte18/loomyard/issues/85).

## Seen

In hub DrillOpPlan (shortname dop), the reed session of beam `loomyard-setup` shows the **task** button group (HALT, CANCEL, code, terminal, agent) instead of the **beam** group, so the distill, refresh, merge-in and land buttons are missing.

- `lym config --resolved reed` resolves `buttons.beam` from the template with distill, refresh, merge-in and land; no default, worktree or local layer writes a `buttons` group (the machine layer does not exist for reed).
- The session's tmux option `@lym_button_spec` holds the task entries with `{slug}` = `loomyard-setup`, and `@lym_button_dir` points at the prime worktree, not the beam worktree.
- The beam's board project shows `running · Beam-Await`, so the beam runs as a batten run; the session seems to be classified by that run as a task session.

## Workaround

None applied; the operator runs `lym orch distill` / `lym orch refresh` by hand or asks the orch.

## Expected

A beam's reed session uses the `beam` group (and its worktree as button dir), whether or not the beam runs under batten.
