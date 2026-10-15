# Card 9 — loom-override-migration

**Create:**
- `internal/loomengine#migrateShedOverrides`
- `internal/loomengine#TestMigrateConfig_ShedOverrides`

**Edit:**
- `internal/loomengine#MigrateConfig`
- `internal/configreg#TestPrepareRunSettings`

**Intent:**
Migrate a hub's `loom.yaml` once, so the four keys the old template spelled out stop masking `shed.yaml` (D5).
Reconcile copies no template comment onto an existing key, so without this every existing hub keeps `llm`, `haiku`, 10 and 4 in `loom.yaml` and the old comments that say empty takes haiku and 0 takes 10.

`migrateShedOverrides` runs inside `MigrateConfig` after its existing steps, editing yaml nodes so every other key and comment is carried, as `migrateProducerKeys` does, in two steps of one pass:
1. Value rewrite: when `driver_kind`, `tackler`, `tackler_timeout_min` and `tackler_max_spawns` hold exactly `llm`, `haiku`, 10 and 4, and `driver_kind`'s line comment is the old template's text for it, the four become the inherit values `""`, `""`, 0 and null, with one report line naming the rewrite.
2. Comment refresh: each of the four keys whose line comment equals the old template's text for that key gets the new template's comment, read from `ConfigTemplate()`'s node for the key, with its value untouched, and one report line per refreshed key.
   A hub that pinned any key gets the new comments without the value rewrite; a comment edited by hand is kept.

The four old comment texts are the retired template's, held as constants beside the migration and named as retired.
Both steps are idempotent: after one pass no key carries an old comment, so a second pass returns the document unchanged with no report lines.
A `run_settings` document carries no comments, so neither step fires there (`configreg`'s `PrepareRunSettings` calls `Migrate`), and a value set anywhere after the migration, the old defaults included, is kept.
A document holding none of the four keys is returned unchanged.
`MigrateConfig`'s doc comment adds the step.

`TestMigrateConfig_ShedOverrides`, a table over whole documents with a second pass on each row's output:
- the old template's four lines with their old comments become the inherit values with the new comments, and the second pass changes nothing and reports nothing;
- the same with `tackler_max_spawns: 0` keeps 0 and every other value, all four keys get the new comments, and the second pass changes nothing;
- old values with `driver_kind`'s comment edited by hand keep every value and that comment, and the other three get the new comments;
- old values with no comments, a `run_settings` document, are returned unchanged;
- a document of new-template lines is returned unchanged.
`TestPrepareRunSettings` gains a row carrying a loom `tackler_max_spawns: 4` through `Migrate` into the loom document as 4.

**ImpactSummary:** loom's config migration gains one comment-keyed step; its only callers are reconcile and run_settings preparation, and the run_settings path is pinned unchanged.
