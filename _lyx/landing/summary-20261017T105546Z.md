# Name review model keys after their roles, one pair per segment

The review segments' model and fan keys now say which role they set, and every review segment, Merge-Review and Darn-Review included, has its own reviewer and fixer model key.

## Key names

| Old | New |
| --- | --- |
| `loom.yaml` `review`, `fix` | `reviewer_model`, `fixer_model` (the shared fallback) |
| `loom.yaml` `<segment>_review`, `<segment>_fix` (Discussion, Plan, Webster) | `<segment>_reviewer_model`, `<segment>_fixer_model` |
| (none) | `loom.yaml` `merge_reviewer_model`, `merge_fixer_model` |
| `loom.yaml` `<segment>_fan`, `<segment>_fix_fan` (Discussion, Plan, Webster, Merge) | `<segment>_reviewer_fan`, `<segment>_fixer_fan` |
| (none) | `darn.yaml` `darn_reviewer_model`, `darn_fixer_model` |
| `darn.yaml` `review_fan`, `review_fix_fan` | `darn_reviewer_fan`, `darn_fixer_fan` |

An empty segment model key takes the shared `reviewer_model` or `fixer_model` in `loom.yaml`, Darn's included.
The other role keys (`judge`, `seed`, the per-segment judge and seed keys, `rework` and the rest) and the non-model `review_*` knobs keep their names.

## Behavior

- Merge-Review runs on `merge_reviewer_model` and `merge_fixer_model` rather than on Webster-Review's resolved models, and Darn-Review on darn.yaml's two new keys.
  The new keys' template defaults equal Webster's (`sonnet[high]` reviewer, empty fixer), so a hub on the template runs both perches on the models they used before.
- A Burler row's refusal or engine error names the key its model came from: darn.yaml's key for a set Darn key, else loom.yaml's shared key.
  `loomengine.SegmentKeys` now carries a `shuttleengine.ModelKey` (module and key) per half, so `reviewRowModelKeys` and the Darn wiring pass the key through without guessing its file.
- `lyx config reconcile` renames every retired key to its new name with its value and the new key's template line comment.
  A document holding both names keeps the new one's value and drops the old.
  A document holding `webster_review` or `webster_fix` and no Merge key gets `merge_reviewer_model` or `merge_fixer_model` set to the same value first, so Merge-Review keeps the models it followed; the report line notes that Darn-Review now reads darn.yaml, which the loom rewrite cannot set.
  The same Migrate hooks run on a task's `run_settings`, so a board entry still naming an old key keeps working.
- `darnengine` gains a `MigrateConfig`, registered in `configreg`, for its two fan-key renames.

## Code

- `burlerengine/modelkeys.go`: `ModelSpecList`, its shape error, `Unset` and `ValidateModelSpecList`, moved from loomengine beside the fan list so darnengine reads the same per-round model grammar (darnengine cannot import loomengine).
- `yamlengine/rename.go`: `RenameKeys`, the top-level key rename both modules' Migrate hooks use.
- `loomengine/review.go`: `ResolveReview` resolves Merge too, and `ResolveDarnReview` resolves Darn's keys over the shared list; `loomcli.wireDarnReview` (formerly `wireDarnFans`) puts the result on the Darn rows.
- The templates, `ConfigSpec` categories and open maps, the recipe comments, `docs/overview.md`, and the package docs of darnengine, burlerengine and yamlengine use the new names.
  The `lyx config` help example now names `reviewer_model`, with shorter values to stay under the help-length cap.

## Design decisions

- Darn's keys carry a `darn_` prefix inside darn.yaml, following the operator's `<segment>_reviewer_model` for every segment, and so they do not share a name with loom.yaml's shared `reviewer_model`.
- The renames go through each module's Migrate hook rather than `configreg.KeyMoves`, because a key move carries one source to one target and cannot also copy Webster's value onto the new Merge key.
- The decision record's acceptance criterion that a strict load refuses an old key does not hold: a base-layer load ignores a key the template lacks, as it does for every retired key.
  The automatic reconcile after a binary change is what moves a hub onto the new names, and the tests pin the rename there instead.
- An empty list (`[]`) on a segment or Darn model key reads as unset and takes the shared list, as loom's segment keys already did; only the shared keys refuse an empty list.

## Tests

Existing config, migration, resolution and wiring tests move to the new names and gain rows for Merge's keys, Darn's model keys and their refusals, the renames with comments and the Merge copy, and the run-settings rename.
`darnengine` gains `TestMigrateConfig`, so its row in `cmd/lyx/testdata/test-budget.yaml` rises to 5.
