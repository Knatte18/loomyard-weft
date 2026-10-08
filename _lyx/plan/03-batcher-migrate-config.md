# Card 3 — batcher-migrate-config

**Create:**
- `internal/batcher#MigrateConfig`

**Edit:**
- `internal/batcher/config_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/batcher/doc.go`

**Uses:**
- `internal/configengine#MarkInvalid`

**Intent:** The batcher's half of D12, in `config.go`: a pure rewrite of a `batcher.yaml` document that removes the retired keys from every entry under `profiles`.
For each profile it drops the profile-level `alone_above`, drops `weights.startup_context`, and drops `weights.master_base`; it adds `weights.orientation` at the template's own value (read from `ConfigTemplate()`'s `cautious` profile, so the number is declared once, in `template.yaml`) only when the profile carries `weights.master_base` or `weights.startup_context` and no `orientation` of its own; an operator's `orientation` is kept.
It returns the rewritten document and one human-readable rewrite per change, such as `profiles.cautious.weights.master_base: removed`, sorted.
A document with no retired key comes back unchanged with no rewrites and no error; an empty document likewise.
Bound: it rewrites only those three keys at their declared level and adds only `orientation` where it is missing; every other entry of every profile, and every key outside `profiles`, is carried byte-for-byte as yaml.v3 node editing allows, with comments kept.
A document that does not parse returns an error wrapped with `configengine.MarkInvalid`'s sentinel through the same unmarshal path the loader uses.
The loader itself never calls it: `Active` keeps refusing a retired key (card 2), and the migration runs only from `lyx config reconcile` (card 4).
`config_test.go` gains `TestMigrateConfig`, one table test over documents, and `test-budget.yaml` raises `internal/batcher` by one for it: a profile with `master_base`, with both `master_base` and `orientation` (orientation kept), with `startup_context`, with `alone_above`, an operator profile with extra keys carried whole, and a clean document; each row asserts the output loads through `Active` without error where the input carried only retired keys.
The package doc names the function and its bound.

**ImpactSummary:** Adds one exported function to batcher's config file; no existing caller changes.
