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
- `internal/yamlengine#TestReconcile`

**Create:**
- `plan:internal/loomengine#validateAdvisorSpecs` -> `func validateAdvisorSpecs(key string, specs ModelSpecList, reg modelspec.Registry) error`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/loomengine#isUnsetModelSpecList`
- `internal/loomengine#ModelSpecList`
- `internal/modelspec#LoadRegistry`

**Intent:** Add the one config key D4 names, `discussion_advisors`, a list of model-specs with one advisor per entry, empty legal, parsed and resolved at load.
`Config` gains `DiscussionAdvisors ModelSpecList` with the yaml tag `discussion_advisors`, so a scalar loads as a list of one as `review` does.
The template gains the line `discussion_advisors: [sonnet[high]]` after `discussion_interactive`, commented as the advisors' model-specs, one advisor per entry, with an empty list running the chair alone; `discussion` stays the chair's model and is unchanged.
`ConfigOpenMaps` lists the key beside the other scalar-or-list model-spec keys, so reconcile and `--set` carry the operator's value whole whatever its spelling.
`LoadConfig`, after unmarshalling, normalises the key before any entry is parsed: an unset list by `isUnsetModelSpecList`, which covers the empty sequence, the empty scalar, null and `[""]` since each unmarshals to no entries or to one empty string, becomes `nil`.
A non-empty list loads the registry through `modelspec.LoadRegistry(baseDir)`, the anchor `wire` loads it from, refusing a registry read failure naming the key with the way forward "fix models.yaml, or set the key to an empty list for no advisors", and `validateAdvisorSpecs` parses and resolves every entry, refusing an empty entry, one `modelspec.Parse` rejects or one the registry does not define, naming the key, the 1-based entry index and the way forward "set the key to a list of model-specs the registry defines, one per advisor, or to an empty list for no advisors".
A value that is neither a scalar nor a sequence of scalars is refused inside `ModelSpecList.UnmarshalYAML` before any entry exists, so `LoadConfig`, when the shape error's key is `discussion_advisors`, returns the key, the value's line and the advisor way forward instead of the review lists' non-empty-list way forward, with no entry index.
Bound: the normalised list's length is the advisor count for every spelling the load accepts; with an empty list the registry is never read; an absent key is filled from the template by `configengine.Load` before the unmarshal, as every loom key is, so "absent" means one advisor in a file and no advisors in an in-memory `Config`.
`templateConfig` gains the default list.
Rows join `TestLoadConfig_Loads` for the key as `""`, null, `[]`, `[""]`, one scalar model-spec and a two-entry list, each asserting the normalised list, and a row for a legacy file without the key taking the template's one advisor.
Rows join `TestLoadConfig_Refuses` for `["", "sonnet[high]"]` and `["opus[effort"]`, each asserting the key, `entry 1` and the advisor way forward, for `["sonnet[high]", "ghost"]` asserting the key, `entry 2`, the unknown alias and the advisor way forward, and for a mapping value asserting the key and the advisor way forward with no entry index.
Rows join `TestReconcile` for an open-map key held as a scalar, as `""` and as null, each carried whole through the merge, the gap beside its existing list row; `TestLookup`'s loom open-maps row gains the key, and together with the load rows they pin that the operator's value survives reconcile.
The refusal spec's loom section gains the row "config: discussion_advisors entry not a model-spec", whose trigger is an empty entry beside another, an entry that is not a model-spec or that the registry does not define, or a mapping value, and whose way forward is the advisor way forward, and a row for the unreadable registry with its way forward.

**ImpactSummary:** One new struct field read only by the table builder of card 5; every existing load keeps its result, the registry is read at load only when the key holds entries, and an existing hub takes the template's one advisor until the operator edits the key.
