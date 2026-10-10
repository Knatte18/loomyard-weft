# Card 3 — loom-config-discussion-advisors

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/configreg#TestLookup`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/loomengine#validateAdvisorSpecs` -> `func validateAdvisorSpecs(key string, specs ModelSpecList) error`
- `plan:internal/loomengine#TestConfig_DiscussionAdvisorsKeptThroughReconcile` -> `func TestConfig_DiscussionAdvisorsKeptThroughReconcile(t *testing.T)`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/loomengine#isUnsetModelSpecList`
- `internal/loomengine#ModelSpecList`

**Intent:** Add the one config key D4 names, `discussion_advisors`, a list of model-specs with one advisor per entry, empty legal.
`Config` gains `DiscussionAdvisors ModelSpecList` with the yaml tag `discussion_advisors`, so a scalar loads as a list of one as `review` does.
The template gains the line `discussion_advisors: [sonnet[high]]` after `discussion_interactive`, commented as the advisors' model-specs, one advisor per entry, with an empty list running the chair alone; `discussion` stays the chair's model and is unchanged.
`ConfigOpenMaps` lists the key beside the other scalar-or-list model-spec keys, so reconcile and `--set` carry the operator's value whole whatever its spelling.
`LoadConfig`, after unmarshalling, normalises the key before any entry is parsed: an unset list by `isUnsetModelSpecList`, which covers the empty sequence, the empty scalar, null and `[""]` since each unmarshals to no entries or to one empty string, becomes `nil`; otherwise `validateAdvisorSpecs` parses every entry and refuses an empty entry or one `modelspec.Parse` rejects, naming the key, the 1-based entry index and the way forward "set the key to a list of model-specs, one per advisor, or to an empty list for no advisors".
Bound: the normalised list's length is the advisor count for every spelling the load accepts; an absent key is filled from the template by `configengine.Load` before the unmarshal, as every loom key is, so "absent" means one advisor in a file and no advisors in an in-memory `Config`.
Registry resolution of each entry stays with the table builder, where `discussion` resolves today, since `LoadConfig` has no registry.
`templateConfig` gains the default list.
Rows join `TestLoadConfig_Loads` for the key as `""`, null, `[]`, `[""]`, one scalar model-spec and a two-entry list, each asserting the normalised list, and a row for a legacy file without the key taking the template's one advisor.
Rows join `TestLoadConfig_Refuses` for `["", "sonnet[high]"]`, `["opus[effort"]` and a mapping value, each asserting the key, `entry 1` or `entry 2` and the way forward.
`TestConfig_DiscussionAdvisorsKeptThroughReconcile` runs `yamlengine.Reconcile` over the template with `ConfigOpenMaps` for a file holding the key as `""`, null, one model-spec, `[]` and a two-entry list, loads the merged bytes through `LoadConfig` and asserts the advisor count is unchanged by the reconcile; it is a new behaviour no load row pins.
`TestLookup`'s loom open-maps row gains the key.
The refusal spec's loom section gains the row "config: discussion_advisors entry not a model-spec", whose trigger is an empty entry beside another, an entry that is not a model-spec, or a mapping value, and whose way forward is the message's, and the existing review-key row's registry clause is extended to name this key, since the table builder resolves its entries at the Discussion-Write call.
The budget file's `internal/loomengine` row rises by one for the new test.

**ImpactSummary:** One new struct field read only by the table builder of card 5; every existing load keeps its result, and an existing hub takes the template's one advisor until the operator edits the key.
