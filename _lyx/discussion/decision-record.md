# Decision record — clearer loom model key names, one per step

## Goal

Every reviewer and fixer model key and every review fan key names the role it sets, and every review segment, Merge and Darn included, gets its own reviewer and fixer model key.

## Scope

- In: loom.yaml's run-wide reviewer and fixer model keys, the per-segment reviewer and fixer model keys of Discussion, Plan and Webster, new ones for Merge, and the per-segment reviewer and fixer fan keys of Discussion, Plan, Webster and Merge.
- In: darn.yaml's new Darn-Review reviewer and fixer model keys, and its two fan keys renamed.
- In: `lyx config reconcile` rewriting the old keys to the new names, in the hub file, local files and a task's `run_settings`.
- In: the docs that name the keys (`doc.go` of the touched packages, `docs/overview.md`, recipe and stencil comments).
- Out: the other role model keys (`discussion`, `plan`, `judge`, `seed`, `*_judge`, `*_seed`, `rework`, `friction`, `driver`, `arbiter`, `tackler`, `alignment`, `writer`) and the non-model `review_*` knobs keep their names.
- Out: `docs/research/` measurement reports, which record past runs under the names they ran with.

## Decisions

### Loom key names

`review` becomes `reviewer_model` and `fix` becomes `fixer_model`, the shared fallback.
`<segment>_review` becomes `<segment>_reviewer_model` and `<segment>_fix` becomes `<segment>_fixer_model` for Discussion, Plan and Webster.
`<segment>_fan` becomes `<segment>_reviewer_fan` and `<segment>_fix_fan` becomes `<segment>_fixer_fan` for Discussion, Plan, Webster and Merge.
Rationale: the operator's 2026-10-17 decision picked flat role names, and a `_model` or `_fan` suffix tells a model key from a fan key and from the `review_*` knobs.

### Merge gets its own model keys

loom.yaml gains `merge_reviewer_model` and `merge_fixer_model`; an empty value takes `reviewer_model` or `fixer_model`, as every other segment's does.
Their template defaults equal Webster's, `sonnet[high]` and empty, so a hub on the template runs Merge-Review on the models it ran on before.
Rationale: today Merge-Review runs on Webster-Review's resolved models with no key of its own.

### Darn gets its own model keys

darn.yaml gains `darn_reviewer_model` and `darn_fixer_model`, and `review_fan` and `review_fix_fan` become `darn_reviewer_fan` and `darn_fixer_fan`.
An empty Darn model key takes loom.yaml's `reviewer_model` or `fixer_model`, the one shared fallback.
Their template defaults equal Webster's, `sonnet[high]` and empty.
A refusal or engine error about a Darn-Review row names darn.yaml's key when that key is set, and loom.yaml's fallback key otherwise.
Rationale: the operator named `<segment>_reviewer_model` for every segment, Darn's in darn.yaml, and a single fallback keeps one place to change every reviewer.

### The per-round model list moves to burlerengine

The one-or-per-round model-spec list type, its shape error and its validation move from loomengine into burlerengine beside the fan list, so darnengine reads its two model keys with the same grammar loom's keys use.
Rationale: darnengine cannot import loomengine, which imports it, and burlerengine already holds the matching per-round fan grammar both modules use.

### Migration through each module's Migrate hook

loom's `MigrateConfig` renames each old key to its new name, keeping the value, and reports one line per rename.
A document holding an old key and its new name already keeps the new name's value, drops the old key and says so.
A document holding `webster_review` or `webster_fix` also gets `merge_reviewer_model` or `merge_fixer_model` set to the same value when it sets neither, so Merge-Review keeps the models it followed, and the report line says Darn-Review now reads darn.yaml's keys.
darn gets a `MigrateConfig` that renames its two fan keys the same way, registered in configreg.
The Migrate hook also runs on a task's `run_settings`, so an entry still naming an old key keeps working.
Bound: the rename carries a value unchanged under its new key and invents no value, except the Merge copy, which only writes a key the document left unset and copies the value of the key Merge read before.
Rationale: reconcile already runs a module's Migrate hook on every layer's file and on run settings, and loom's retirements live there.

## Constraints

- `PATTERN-config-layers`: only configengine reads and stacks layers; the migration edits the bytes it is handed.
- `PATTERN-tolerant-records`: not touched; config files stay strict through configengine.
- `PATTERN-refusal-way-forward`: every refusal naming a renamed key names the new key and its way forward.
- `PATTERN-comment-line-breaks`: Go comments one sentence per line.
- `PATTERN-test-economy`: renamed keys extend the existing config and migrate tests; new tests only for the new Merge and Darn resolution and the darn migration.

## Auto-mode assumptions

- Darn's keys carry the `darn_` prefix inside darn.yaml, read literally from the operator's `<segment>_reviewer_model` for every segment.
- The fan keys take `_reviewer_fan` and `_fixer_fan`, "the same style" as the model keys.
- The new model keys are category none, like the keys they replace, and the renamed fan keys stay guard.

## Open risks

- A hub whose loom.yaml overrode `webster_review` or `webster_fix` sees Darn-Review move to darn.yaml's template default; reconcile cannot copy across files, so its report line names darn.yaml's keys for the operator to set.
- A board entry's `run_settings` keeps the old key names on disk; the Migrate hook rewrites them at each read.

## Acceptance criteria

- loom.yaml and darn.yaml load under the new names, and the old names are refused as unknown keys by a strict load.
- Each segment's reviewer and fixer, Merge and Darn included, runs on its own key when set and on the shared fallback when empty.
- A refusal or engine error on a segment's model names the key its value came from, darn.yaml's for a set Darn key.
- `lyx config reconcile` rewrites every old key to its new name with its value, keeps a new name already set, and copies Webster's values onto unset Merge keys; a second pass changes nothing.
- A task's `run_settings` naming an old key is rewritten to the new name.
- The darn migration renames `review_fan` and `review_fix_fan`.

## Notes for the plan writer

- `reviewRowModelKeys` and `wireDarnFans` in `internal/loomcli` name each Burler row's model and fan keys; Merge and Darn rows currently borrow Webster's.
