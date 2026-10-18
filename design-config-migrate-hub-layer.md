# Key-rename migration never reaches the hub config: loom-model-key-names silently turned off the hub fans

`config-migrate-hub-layer` · Task · bug, config, loom · running · Darn-Bouncer

Imported from [issue #709](https://github.com/Knatte18/loomyard/issues/709).

## Seen

After `loom-model-key-names` landed (`a273d69bc`), the hub's `_board/_lyx/config/loom.yaml` and `darn.yaml` still held the old key names (`review`, `discussion_review`, `webster_fan`, `merge_fix_fan`, darn `review_fan`, ...).
The new keys resolved from the template, so every run silently lost the hub's lens fan, twill fixer and reviewer models: `lyx config loom --resolved` showed `webster_reviewer_fan: "" # template` and the old keys as `# default`.

- `lyx config reconcile` in the prime reports loom with no `migrated` entry: it never reaches the hub default layer.
- `lyx config reconcile` (dry-run) run from `_board/` does list the loom renames, but treats the hub files as a worktree layer: it would strip most keys and mark `darn`, `gate`, `landing`, `language`, `fabric` and `board` as retired, so `--apply` there is unsafe.
- Nothing at landing or deploy ran the migration, and nothing warned that the hub file held unknown keys.

## Workaround

The orch carried every value to its new name by hand with `lyx config loom --layer default --set ...` and `lyx config darn --layer default --set ...`.
The old keys stay in both files as `preserved` entries, since `--set` cannot remove a key.

## Expected

A key-rename migration reaches the hub's default layer, through a hub-scoped `lyx config reconcile` or at landing, and removes the old keys.
A hub config file holding keys the template no longer knows is reported loudly at load.
## Operator decision (2026-10-18)

The task also removes the `preserved` old keys that `loom-model-key-names` left in the hub's `loom.yaml` and `darn.yaml`, through the migration it builds.
