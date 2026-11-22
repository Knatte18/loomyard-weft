# Per-task beam settings use one lookup from the worktree up

`per-task-settings-one-lookup` · Note · enhancement, dobby, config, board, landing

## What was seen (2026-11-22)

Three `beam.yaml` settings have a beam value and a per-task value, each wired its own way:

- `pull_request`: the task's value is a field on its board entry, and the beam's mode decides how far it counts.
- `land_notice`: the task's value is the board field `notify_on_land`, and it counts only when the beam says `per-task`.
- `autonomy` (task `orch-autonomy-mode`): the task's value is set in `run_settings`, copied into its worktree like every other setting, and looked up from the worktree up the chain to the hub default.

## Idea (operator, 2026-11-22)

Move `pull_request` and `land_notice` onto the same rule as `autonomy`:
the task's value in `run_settings`, and one lookup from the task's worktree up to the hub, where the nearest worktree that sets the key wins in both directions.
The board fields `pull_request` and `notify_on_land`, and the beam modes that only exist to defer to them, such as `land_notice: per-task`, then go.
One lookup in place of three wirings should make the code smaller.

It builds on the lookup `orch-autonomy-mode` adds, so it runs after that task lands.
A meaning change of a config key ships the reinterpretation `PATTERN-config-layers` asks for, and the existing board fields are carried over to `run_settings` once.
