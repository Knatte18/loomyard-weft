# Card 3 — loom-config-discussion-keys

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
- `plan:internal/loomengine#DiscussionProducerSingle` -> `const DiscussionProducerSingle = "single"`
- `plan:internal/loomengine#DiscussionProducerSeats` -> `const DiscussionProducerSeats = "seats"`
- `plan:internal/loomengine#validateAdvisorSpecs` -> `func validateAdvisorSpecs(key string, specs ModelSpecList, reg modelspec.Registry) error`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/loomengine#isUnsetModelSpecList`
- `internal/loomengine#ModelSpecList`
- `internal/modelspec#LoadRegistry`

**Intent:** Add the two `loom.yaml` keys the design needs: `discussion_producer`, which selects the producer the Discussion-Write row runs (the parent's decision superseding D1), and `discussion_advisors`, the seat-table producer's advisor list (D4).
`Config` gains `DiscussionProducer string` with the yaml tag `discussion_producer` and `DiscussionAdvisors ModelSpecList` with the yaml tag `discussion_advisors`, so the advisor scalar loads as a list of one as `review` does.
The two producer values are the exported constants `DiscussionProducerSingle` and `DiscussionProducerSeats`, which `internal/loomcli` compares the loaded value against in card 8; `LoadConfig` refuses any other value naming the key, the value and both legal values, as it refuses an unknown `fix_start`.
The template gains, after `discussion_interactive`, the line `discussion_producer: single`, commented as the Discussion-Write row's producer: `single` runs the single-agent writer, `seats` runs a chair with the advisors below; then the line `discussion_advisors: ["sonnet[high]"]`, commented as the advisors' model-specs, one advisor per entry, read only when `discussion_producer` is `seats`, with an empty list running the chair alone.
The entry is quoted because `[` is a flow indicator inside a YAML flow sequence; `discussion` stays the chair's model and is unchanged.
`ConfigOpenMaps` lists `discussion_advisors` beside the other scalar-or-list model-spec keys, so reconcile and `--set` carry the operator's value whole whatever its spelling; `discussion_producer` is a plain scalar and needs no open-map entry, since reconcile keeps a scalar's user value.
`LoadConfig`, after unmarshalling, normalises `discussion_advisors` before any entry is parsed: an unset list by `isUnsetModelSpecList`, which covers the empty sequence, the empty scalar, null and `[""]` since each unmarshals to no entries or to one empty string, becomes `nil`.
A non-empty list loads the registry through `modelspec.LoadRegistry(baseDir)`, the anchor `wire` loads it from, refusing a registry read failure naming the key with the way forward "fix models.yaml, or set the key to an empty list for no advisors", and `validateAdvisorSpecs` parses and resolves every entry, refusing an empty entry, one `modelspec.Parse` rejects or one the registry does not define, naming the key, the 1-based entry index and the way forward "set the key to a list of model-specs the registry defines, one per advisor, or to an empty list for no advisors".
The list is validated whatever `discussion_producer` holds, so a mistake fails at load rather than when the operator first switches producers.
A value that is neither a scalar nor a sequence of scalars is refused inside `ModelSpecList.UnmarshalYAML` before any entry exists, so `LoadConfig`, when the shape error's key is `discussion_advisors`, returns the key, the value's line and the advisor way forward instead of the review lists' non-empty-list way forward, with no entry index.
Bound: the normalised list's length is the advisor count for every spelling the load accepts; with an empty list the registry is never read; an absent key is filled from the template by `configengine.Load` before the unmarshal, as every loom key is, so "absent" means one advisor in a file and no advisors in an in-memory `Config`, and an absent `discussion_producer` means `single`.
`templateConfig` gains both defaults.
Rows join `TestLoadConfig_Loads` for `discussion_advisors` as `""`, null, `[]`, `[""]`, one scalar model-spec and a two-entry list, each asserting the normalised list, a row for a legacy file without either key taking `single` and the template's one advisor, and a row for `discussion_producer: seats`.
Rows join `TestLoadConfig_Refuses` for `discussion_advisors` as `["", "sonnet[high]"]` and `["opus[effort"]`, each asserting the key, `entry 1` and the advisor way forward, for `["sonnet[high]", "ghost"]` asserting the key, `entry 2`, the unknown alias and the advisor way forward, for a mapping value asserting the key and the advisor way forward with no entry index, and for `discussion_producer: sideways` asserting the key, `"single"` and `"seats"`.
Rows join `TestReconcile` for an open-map key held as a scalar, as `""` and as null, each carried whole through the merge, the gap beside its existing list row; `TestLookup`'s loom open-maps row gains `discussion_advisors`, and together with the load rows they pin that the operator's value survives reconcile.
The refusal spec's loom section gains the rows "config: discussion_advisors entry not a model-spec", whose trigger is an empty entry beside another, an entry that is not a model-spec or that the registry does not define, or a mapping value, and whose way forward is the advisor way forward; "config: models.yaml unreadable for discussion_advisors" with its way forward; and "config: unknown discussion_producer", whose way forward is to set the key to `single` or `seats` in loom.yaml.

**ImpactSummary:** Two new struct fields read only by the table builder of card 5 and the wiring of card 8; every existing load keeps its result, the registry is read at load only when the advisor key holds entries, and an existing hub runs the single-agent producer with the template's one advisor recorded until the operator edits the keys.
