# Name loom model keys by role, with one reviewer and fixer per step

Loom's model keys now say what they set, and every review segment has its own reviewer and fixer model key.
`fix` read like a flag; it is now `fixer_model`.
Merge-Review and Darn-Review used to borrow Webster's models and now have keys of their own.

## Renames

In `loom.yaml`:

- `review` and `fix` become `reviewer_model` and `fixer_model`, the shared fallback an empty segment key takes.
- `discussion_review`, `plan_review` and `webster_review` become `<segment>_reviewer_model`; the `_fix` keys become `<segment>_fixer_model`.
- `discussion_fan`, `plan_fan`, `webster_fan` and the Merge fan become `<segment>_reviewer_fan`; the `_fix_fan` keys become `<segment>_fixer_fan`.
- New: `merge_reviewer_model` and `merge_fixer_model`, defaulting to Webster's template values (`sonnet[high]` and empty), so a hub on the template runs Merge-Review on the models it ran on before.

In `darn.yaml`:

- New: `darn_reviewer_model` and `darn_fixer_model`, defaulting to Webster's template values.
  An empty value takes `loom.yaml`'s `reviewer_model` or `fixer_model`.
- `review_fan` and `review_fix_fan` become `darn_reviewer_fan` and `darn_fixer_fan`.

The other role model keys (`judge`, `seed`, `rework`, `friction`, `driver`, `arbiter`, `tackler`, `alignment`, `writer`) and the non-model `review_*` knobs keep their names.
A strict load refuses the old names as unknown keys.
A refusal or engine error about a segment's model names the key the value came from, `darn.yaml`'s key for a set Darn key.

## Migration

`lyx config reconcile` renames each old key to its new name with its value, in the hub file, local files and a task's `run_settings`, and reports one line per rename.
A document that already holds the new name keeps it, drops the old key and says so.
A second pass changes nothing.
`darnengine` gets a `MigrateConfig` for its two fan keys, registered in `configreg`.

A document that sets `webster_review` or `webster_fix` and neither Merge key gets `merge_reviewer_model` or `merge_fixer_model` set to the same value, so Merge-Review keeps the models it followed.
This is the only value the migration writes that the document did not already hold.

## Design decisions

- `yamlengine.RenameKeys` is the shared top-level rename.
  It keeps each value, every other key and the comments, and gives the renamed key the template's line comment.
- The one-or-per-round model-spec list type, its shape error and its validation moved from `loomengine` into `burlerengine` beside the fan list.
  `darnengine` cannot import `loomengine`, and now reads its two model keys with the same grammar.
- `loomcli` names each Burler row's model and fan keys in `reviewRowModelKeys` and `wireDarnFans`; Merge and Darn rows no longer borrow Webster's.
- Docs follow: `doc.go` of the touched packages, `docs/overview.md`, and the recipe comments.
  `docs/research/` reports keep the names their runs used.

## To check by hand

A hub whose `loom.yaml` overrode `webster_review` or `webster_fix` sees Darn-Review move to `darn.yaml`'s template default.
Reconcile cannot copy across files, so its report line names the `darn.yaml` keys for the operator to set.
A board entry's `run_settings` keeps the old key names on disk; the migration rewrites them at each read.
