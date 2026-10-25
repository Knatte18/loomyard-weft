# Card 4 — Consumers outside fabricengine use each worktree's own anchor

**Edit:**
- `internal/hubgeom#DobbyGeometry`
- `internal/pairteardown#Teardown.taskAnchor` -> `func (t *Teardown) taskAnchor(slug string) (string, error)`
- `internal/pairteardown#Teardown.busyError`
- `internal/pairteardown#Teardown.archivePairLogs`
- `internal/pairteardown#pairWithLogs`
- `internal/ideengine#Spawn`
- `internal/ideengine#OpenTask`
- `internal/ideengine#writeWorktreeConfig`
- `internal/ideengine#Menu`
- `internal/fabriccli#runReconcile`
- `internal/hubgeom#TestDobbyGeometry`
- `internal/ideengine#TestSpawnAnchoredScenario`
- `internal/ideengine#TestMenuScenario`
- `internal/pairteardown_test#TestRun_TeardownScenario`

**Create:**
- `internal/fabriccli_test#TestRunCLI_ReconcileFromAPairAtAnotherAnchor`

**Uses:**
- `internal/lyxcwd#NamedWorktree`
- `internal/hubforge#ReanchorPair`
- `internal/ideengine#resolveSpawnTarget`

**Intent:**
D3 at the consumers that join a slug-built path with the prime's or the acting worktree's `AnchorRel`.
`DobbyGeometry` places a beam's `.lyx/beam/` directory under the beam's own anchor, from `lyxcwd.NamedWorktree(prime.HubPath, worktree)`; for the prime it keeps the prime Location it is handed. A `NamedWorktree` error is returned naming the worktree.
`Teardown.taskAnchor` joins the task worktree with the task's own anchor from `NamedWorktree` and returns that read's error: `busyError` returns it, wrapped naming the pair, in place of the refusal, and `archivePairLogs` warns and archives nothing, as it does when listing the logs fails.
In production `busyError`'s read never fails, since `probeQuiet`'s `ResolveWorktree` of the same marker has already succeeded; `pairWithLogs` fails its test on the error.
`ideengine.Spawn`, `OpenTask` and `writeWorktreeConfig` open and configure a task at the task's own anchor (`openDir`, the `.vscode/tasks.json` path, the anchored `info/exclude` entry and `vscode.WriteConfig`'s anchor) from `NamedWorktree(l.HubPath, slug)`; `Menu` probes each listed worktree's `_lyx` at that worktree's own anchor.
`runReconcile` takes `primeBaseDir` from the prime's own Location (`NamedWorktree(l.HubPath, primeName)`), so a reconcile run from a pair at another anchor seeds hub config from the prime's real base directory.

Tests:
`TestDobbyGeometry` hand-builds a prime at `AnchorRel` `sub` over a temp hub with no marker, so its existing `a-beam` row, which expects the beam's paths under `sub`, now writes a `.lyx-anchor` of `sub` into the beam's records sibling directory to keep that expectation; a new beam row whose records sibling holds `other` gets its lock, state and marker paths under `<hub>/<beam>/other/.lyx/beam/`, while the prime row stays at the prime's `sub`.
`TestSpawnAnchoredScenario` gains a step with a task re-anchored by `hubforge.ReanchorPair` at another directory than the hub's: `SeedTask` writes `.vscode/` at the task's anchor and `OpenTask`'s launched folder is `<hub>/<slug>/<its anchor>`.
`TestMenuScenario` gains a row with one listed worktree whose `_lyx` sits only under its own anchor `sub` (its records sibling holding a `.lyx-anchor` of `sub`): the menu lists it, which a probe at the prime's anchor would not.
`TestRun_TeardownScenario` gains a step: a live pair re-anchored at `sub` by `hubforge.ReanchorPair` whose driver lock, taken by the test at the pair's own anchor, is held, refuses with `ErrDriverBusy` naming `lyx reed attach, run from <hub>/<slug>/sub`, and the lock-held refusal names the lock path under `<slug>/sub/`.
`TestRunCLI_ReconcileFromAPairAtAnotherAnchor` (tier1, hubforge): with the prime at `.` and a pair re-anchored at `sub`, the prime's own worktree config holds a non-template unit of one module, and that module's hub-wide file is removed from the board; `lyx fabric reconcile` run from the pair's anchored directory reseeds the file carrying that unit, read from the prime's base directory at `.`, where a read at `<prime>/sub` finds nothing and seeds the template only.

**ImpactSummary:** `Teardown.taskAnchor` gains an error result with its two production callers and one test helper in `pairteardown`; seven functions across `hubgeom`, `pairteardown`, `ideengine` and `fabriccli` read the target worktree's own anchor, identical to today's value in a one-anchor hub.
