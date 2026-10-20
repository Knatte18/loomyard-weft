# Card 1 — yamlengine per-name maps

**Edit:**
- `internal/yamlengine#Reconcile` -> `func Reconcile(template, existing []byte, perNameMaps []string, openMaps ...string) (merged []byte, added, removed, kept []string, err error)`
- `internal/yamlengine#Prune` -> `func Prune(template, existing []byte, perNameMaps []string, openMaps ...string) (pruned []byte, removed, kept []string, err error)`
- `internal/yamlengine#SetValues` -> `func SetValues(template, existing []byte, pairs []KV, perNameMaps []string, openMaps ...string) (SetResult, error)`
- `internal/yamlengine#SetSparse` -> `func SetSparse(template, existing []byte, pairs []KV, perNameMaps []string, openMaps ...string) (SetResult, error)`
- `internal/yamlengine#knownKeys` -> `func knownKeys(templateLeaves, templateSequences map[string]*yaml.Node, openMaps, perNameMaps []string) []string`
- `internal/yamlengine#unknownKeys` -> `func unknownKeys(pairs []KV, templateLeaves, templateSequences map[string]*yaml.Node, openMaps, perNameMaps []string) []string`
- `internal/yamlengine#unitRule`
- `internal/yamlengine#unitRule.pruneMapping` -> `func (rule unitRule) pruneMapping(template, existing *yaml.Node, path string, removed, kept *[]string)`
- `internal/yamlengine/doc.go`
- `internal/configsync#reconcileBaseWorktreeFile`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#reconcileSparseFile`
- `internal/configengine#warnUnknownKeys`
- `internal/configengine#Set`
- `internal/configengine#setSparse`
- `internal/yamlengine#TestReconcile`
- `internal/yamlengine#TestPrune`
- `internal/yamlengine#TestSetValues`
- `internal/yamlengine#TestSetValues_UnknownKeys`
- `internal/yamlengine#TestSetValues_Refusals`
- `internal/yamlengine#assertMergedKeyValue`
- `internal/yamlengine#TestSetSparse`

**Intent:** Add a per-name map path kind beside open maps, for D2.
A per-name map is a dotted path P whose template value is a mapping whose keys are names, each name one unit carried whole: a lens's prose scalar or a fan's list.

- `Reconcile` gains `perNameMaps` and a `kept` result.
  For each P the merged mapping holds the template's names in template order, each with the file's value whole when the file holds that name, then every name the file holds and the template lacks, appended in file order with its value whole.
  `added` reports `P.<name>` for each template name the file lacks, and every template name when the file lacks P;
  `kept` reports, sorted, `P.<name>` for each name the file holds and the template lacks;
  nothing at or under P is reported removed, and a list under a name is carried whole, never merged element by element.
  A file whose value at P is null or an empty mapping is an empty per-name map: every template name is added and nothing is kept.
  A file whose value at P is any other non-mapping is refused with an error naming P and the shape it holds, reached by a `TestReconcile` row whose existing holds `lenses: [a]` against a mapping template.
- `Prune` gains `perNameMaps` and a `kept` result: under P every name stays whole, and a name the template lacks is reported in `kept` as `P.<name>`, never in `removed`; `unitRule` carries the per-name paths and `unitRule.pruneMapping` honours them, collecting the kept names through its new `kept` parameter.
- `SetValues` and `SetSparse` gain `perNameMaps`: `P.<name>` is known for any one dot-free name, `knownKeys` lists `P.<name>` for each P, and `unknownKeys` refuses as unknown a key with a further dotted segment under P and the key P itself.
  The pair sets the name under P to the shape its value spells through `scalarOrListNode`, a value starting with `[` a flow list replacing the entry whole and any other value a string, appending the name when absent; a bracketed value that is no such list is refused by `scalarOrListNode`'s error, and nothing is written.
  A null at P becomes an empty mapping before the name is appended, as an open map's does.
  `SetValues` first carries every name existing holds under P and the template lacks onto the working tree's mapping at P, so a hub-only name survives a set.
- An empty `perNameMaps` keeps today's behaviour byte for byte; every production caller passes `nil` here, and cards 2 and 4 pass the module's declared paths.
  The callers in `configsync` take the new `kept` results as unused until card 4.
- `doc.go` documents the per-name kind in its own section beside "Open maps".

Rows, each in the named table: `TestReconcile` — a template name the file lacks is added as `P.<name>`, a file-only name is kept and reported in `kept` with its value, a tuned value and a shorter list under a template name are kept unchanged, a file lacking P reports every template name added, a null P reports every template name added and keeps none, and the non-mapping refusal;
`TestPrune` — a name under P the template lacks stays and is reported kept;
`TestSetValues` — a new name set to a string and a new name set to `[a, b]` are appended, and a hub-only name in existing survives an unrelated set;
`TestSetValues_UnknownKeys` — `P.<name>.x` and `P` itself are unknown and `P.<name>` is known;
`TestSetValues_Refusals` — `P.<name>=[unterminated` is refused naming the key;
`TestSetSparse` — a new name under P is written alone into the sparse file.
The existing rows pass `nil` for `perNameMaps`.

Harness changes the rows need:
`TestReconcile`'s table gains `perNameMaps`, `wantKept` and `wantErr` fields; a `wantErr` row asserts the error names its text and skips the idempotency pass, and every other row's second pass asserts the same `kept` as the first and no `added`.
`TestPrune`'s template gains a `names:` mapping holding one name, its call passes `[]string{"names"}` as `perNameMaps`, and the table gains a `wantKept` field, empty for the existing rows.
`TestSetValues`, `TestSetValues_UnknownKeys`, `TestSetValues_Refusals` and `TestSetSparse` gain a `perNameMaps` field passed to the call.

**ImpactSummary:** Four exported yamlengine functions are re-signed; their production callers are the six configsync and configengine members listed, which pass nil, so no behaviour changes outside yamlengine's own tests.
