# Card 21 — Seed-Child run settings

**Create:**
- `plan:internal/battenshed#ErrInvalidRunSettings` -> `var ErrInvalidRunSettings = errors.New("battenshed: the board task's run settings do not validate")`
- `plan:internal/battenshed#TestSeedChild_RunSettings` -> `func TestSeedChild_RunSettings(t *testing.T)`
- `plan:internal/battencli#stepSeedChild_WritesTheRunSettingsIntoThePairsLocalLayer` -> `func stepSeedChild_WritesTheRunSettingsIntoThePairsLocalLayer(t *testing.T, h *hubforge.Hub)`

**Edit:**
- `internal/battenshed#SeedChildDeps`
- `internal/battenshed#seedChildProducer.Call`
- `internal/battenshed#newSeedChildDeps`
- `internal/battenshed/doc.go`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/battencli#TestBattenIntegration_Rows`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/configreg#PrepareRunSettings`
- `internal/configengine#LocalFile`
- `internal/fabricengine#ConfigDirs`

**Intent:**
Make batten's Seed-Child row write the pair's local overrides from the board task's `run_settings` (D9), so a small task's settings need no hand step and never show up as a tracked change at landing.

`SeedChildDeps` gains four injected closures, wired in battencli with the pair anchor handed in, as `ReadBoardType` is, so `internal/battenshed`'s import allowlist stays unchanged (`PATTERN-told-geometry`):
`ReadRunSettings(ctx)` returns the task's `run_settings`, read fresh from the board on every Call;
`PrepareRunSettings(settings)` returns each named module's migrated local document, an error wrapping `ErrInvalidRunSettings` meaning a key does not validate;
`WriteLocalOverride(ctx, module, doc)` writes one `<pair>/.lyx/config/<module>.local.yaml` atomically;
`RemoveLocalOverrides(ctx, keep)` removes the local file of every registered module not in `keep` and returns the removed paths.

`seedChildProducer.Call` runs them after `ChildDriver` and before `WriteSeed`: read, prepare, write each named module, then remove the rest, logging one Warn per removed file, so the pair's local layer is exactly the task's run settings.
An unreadable board is Stuck as for the type; an error wrapping `ErrInvalidRunSettings` routes to Stuck with the reason naming the module and the key and the way forward: correct the task's `run_settings` with `lyx board upsert`, then `lyx batten run <slug>`; any other prepare, write or remove failure is a hard error.
The input that reaches the Stuck branch after every upstream check is a board entry whose `run_settings` were valid when written but no longer validate at Seed-Child, such as a key a later binary retired or closed to the local layer, or one written to `board.json` by hand.
A resumed Seed-Child redoes the whole write from the current board entry, and no other row and no child launch writes the files; the only local files it can find are ones written while it stood Stuck, which the resume discards (D9's bound).
Batten writes no seed field for run settings (`PATTERN-shed-run-directory`).

In `battenCLI.wire`: `ReadRunSettings` opens the board over `fabricengine.BoardDir(location.HubPath)` as `ReadBoardType` does; `PrepareRunSettings` calls `configreg.PrepareRunSettings` and wraps a `*configreg.RunSettingsError` in `battenshed.ErrInvalidRunSettings`; `WriteLocalOverride` and `RemoveLocalOverrides` resolve the task location lazily and use `configengine.LocalFile(taskLocation.AnchorPath(), module)` over `configreg.Names()`, creating the directory as needed.

`contracts/specs/refusal-spec.md`'s shed section gains the row "Seed-Child: run settings invalid", a correctness halt whose way forward is "correct the task's `run_settings` with `lyx board upsert`, then `lyx batten run <slug>`", and the out-of-scope paragraph names it beside the other admitted rows.
`doc.go` of battenshed describes the write, the removal, the Stuck routing and the bound.

`TestSeedChild_RunSettings` is table-driven over stub closures: named modules are written and the others removed before `WriteSeed`; empty or absent settings remove every local file; an invalid key is Stuck naming module, key and the way forward with no seed written; a failing `ReadRunSettings` is Stuck with no seed written; a write failure is a returned error; a Call interrupted between its writes and its removals and then run again leaves the local layer equal to the board's current settings.
`newSeedChildDeps` fills the new closures with recording stubs.
`stepSeedChild_WritesTheRunSettingsIntoThePairsLocalLayer`, run from `TestBattenIntegration_Rows`, drives Worktree-Create and Seed-Child over a hubforge hub whose task carries `{"loom": {"review_max_bounces": 2}}`: the pair's `loom.local.yaml` holds the key, `loomengine.LoadConfig(fabricengine.ConfigDirs(pair), "loom")` resolves a budget of 2, and `fabricengine.ReadConfigChanges` over the per-worktree config files reports no change.
The same step drives a second task whose `board.json` entry carries `{"darn": {"verify": "x"}}`, written straight into the board file and bypassing `lyx board upsert`: Seed-Child stands Stuck with a reason naming `darn`, `verify` and the way forward, and no child seed and no local file are written, which reaches battencli's wrap of `*configreg.RunSettingsError` in `battenshed.ErrInvalidRunSettings`.
`TestWire_EverySeamFilled` covers the new seams.
The budget row for `internal/battenshed` rises by one.

**ImpactSummary:** four new Seed-Child seams and one Stuck route: battencli wires them, the refusal spec gains one shed row, and no other row changes.
