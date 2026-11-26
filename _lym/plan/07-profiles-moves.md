# Card 7 — profiles-moves

**Create:**
- `internal/configreg#MoveSettings`
- `internal/configreg#TestMoveSettings`
- `internal/configsync/testdata/profiles-flat.yaml`

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`
- `internal/hubreconcile_test#TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`

**Extends:**
- `internal/configreg/doc.go`

**Uses:**
- `internal/yamlengine#MoveKeys`
- `internal/configengine#KeyName`

**Intent:** Reconciling `profiles.yaml` moves the loom and tapestry keys of every recipe block before the template follow (D6), through the configreg helper D7 also uses.

`configreg.MoveSettings(document, prefix)` treats the mapping at `prefix` (the root when empty) as a module-keyed settings object.
For each registered module whose spec has `MovedKeys` it calls `yamlengine.MoveKeys` under `<prefix>.<module>` and returns the moved bytes and one line per change, naming both full names through `KeyName`, such as `moved loom.code_max_bounces to loom.code.max_bounces`, with D5's drop lines.
It keeps comments and every key it does not move; a document with no old key comes back unchanged.

`configreg.Module` gains `RunSettingsBlocks bool`, set for the run-profile module alone in `Modules()`: its file's second-level mappings, `<profile>.<recipe>`, each hold a module-keyed settings object.
`ReconcileHubWideAt`, for a seed-only module with the flag and a present file, moves every `<top>.<second>` mapping of the file and of its template base through `MoveSettings` before `yamlengine.FollowTemplate`;
the follow then compares file, base and template at the new paths, so a shipped block the operator never changed takes the new template, and a shipped value the operator changed stays theirs.
The lines join the result's `Moved`, prefixed with `profiles.yaml` and the block's `<profile>.<recipe>`, so card 6's walk mails them with D5's lines.
The file is written when its bytes after the move and the follow differ from the bytes read, so an untouched flat file is rewritten nested even when the follow reports no unit change.
A `defaults` entry's scalar values hold no mapping, so `MoveKeys` passes them over.
It is idempotent and needs no marker, and `lym config reconcile --layer default --apply` runs it too, since that command reaches `ReconcileHubWideAt`.

`internal/configsync/testdata/profiles-flat.yaml` is a committed copy of the profiles template as it stood before card 4, flat keys and all, the old file and template base the tests start from.

Tests:
- `TestMoveSettings` (tier0, new): a module-keyed document `{loom: {code_max_bounces: 3}, drift: {scope: full}}` comes back `{loom: {code: {max_bounces: 3}}, drift: {scope: full}}` with one line naming both full names; a prefix moves only below it.
- `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` (tier0) rows, each with `profiles-flat.yaml` as both file and template base:
  the untouched file parses, after the reconcile, to the same YAML value as the new template;
  a profile the operator added with `tapestry: {loom: {code_max_bounces: 3}}` holds `loom: {code: {max_bounces: 3}}`;
  a file whose shipped `solid.tapestry` block the operator changed to `loom` `code_max_bounces: 3` holds `loom` `code.max_bounces: 3`.
- `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` (tier1): a hub whose `profiles.yaml` holds a profile the operator added with a flat loom key ends the walk with the key nested, and the mailed notices name the `profiles.yaml` move line.

`configreg/doc.go` gains a sentence naming `MoveSettings` and the `RunSettingsBlocks` flag.

**ImpactSummary:** One new configreg helper and Module flag set for the profiles module alone, read by configsync's seed-only branch; no other module's reconcile changes.
