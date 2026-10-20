# Card 4 — configsync per-name reconcile

**Edit:**
- `internal/configsync#Result`
- `internal/configsync#reconcileBaseWorktreeFile`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#reconcileSparseFile`
- `internal/configcli#reconcileEnvelope`
- `internal/hubreconcile#logResult`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`
- `internal/configsync#TestReconcileAll_SparseLayers`
- `internal/configsync#TestReconcileAll`
- `internal/configcli#TestConfigReconcileInGitRepo`

**Uses:**
- `plan:internal/lenses#ConfigTemplate`
- `internal/configsync#writeModuleFile`
- `internal/configsync#findResult`

**Intent:** Reconcile honours a module's per-name maps at every layer (D2).
`Result` gains `KeptExtra []string`: sorted, each `<path>.<name>` a per-name map of the file holds and the template lacks, kept in place and never pruned; set on a dry run too.
`reconcileBaseWorktreeFile` and `ReconcileHubWideAt` pass `m.PerNameMaps` to `yamlengine.Reconcile` and set `KeptExtra` from its `kept`, so a base file gains each template lens or fan it lacks, reported in `Added` per name, keeps its tuned prose and fan lists, and keeps a lens or fan the template lacks, reported in `KeptExtra` and never in `Removed`.
`reconcileSparseFile` passes `m.PerNameMaps` to `yamlengine.Prune` and sets `KeptExtra` from its `kept`, so a sparse layer keeps a hub-only lens or fan the same way and adds nothing.
A file holding only kept extras needs no write: the write conditions stay on added, removed and migrated keys.
`reconcileEnvelope` renders a non-empty `KeptExtra` as `keptExtra`, omitted when empty, as it does `keptTarget`.
`hubreconcile`'s `logResult` logs `keptExtra` beside `keptTarget`, so a hub-only lens or fan a walk keeps appears in the walk's log of a changed module.

Rows: `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains `wantAdded` and `wantKeptExtra` fields, checked against the applied result when set, and a `lenses` row whose present board default is the template with one lens and one fan removed, one lens's prose tuned, the `standard` fan shortened and one hub-only lens and fan added (built in the test from `lenses.ConfigTemplate()`):
the applied file holds the removed lens and fan again, keeps the tuned prose, the shortened fan and both hub-only entries, `wantAdded` names `lenses.<removed>` and `fans.<removed>`, and `wantKeptExtra` names the two hub-only entries.
`TestReconcileAll_SparseLayers` gains a `wantKeptExtra` field and a `lenses` row whose worktree file holds one hub-only lens: the file stays byte-identical, nothing is added or removed, and `KeptExtra` names the lens.
`TestReconcileAll` gains a `wantKeptExtra` field and two standalone `lenses` rows, the base worktree file that `reconcileBaseWorktreeFile` reconciles, each file built from `lenses.ConfigTemplate()`:
a dry run over a file lacking one template lens and holding one hub-only lens reports the lens added and the hub-only lens in `KeptExtra`, and writes nothing;
an apply over the template plus one hub-only lens reports it in `KeptExtra`, nothing added or removed, is not applied and leaves the file byte-identical.
`TestConfigReconcileInGitRepo` seeds a `lenses.yaml` holding a hub-only lens and asserts the `lenses` entry of the reconcile envelope carries `keptExtra` naming it, and that an entry with nothing kept carries no `keptExtra` key.

**ImpactSummary:** Only the lenses module declares per-name maps, so only lenses.yaml files reconcile differently; every other module passes an empty list and keeps today's results, the envelope gains one optional field, and the hub walk's log of a changed module gains `keptExtra`.
