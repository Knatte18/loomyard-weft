# Card 6 — The parity package, its seat catalog and the per-seat parity test

**Create:**
- `internal/shuttleparity/doc.go`
- `plan:internal/shuttleparity#UnrunHooks` -> `func UnrunHooks(run shuttleengine.HookRun) ([]byte, error)`
- `plan:internal/shuttleparity#ConfiguredSandbox` -> `func ConfiguredSandbox(bin, home, dir string) error`
- `plan:internal/shuttleparity#PromptItems` -> `func PromptItems(stencilsDir, engine, prompt string) []shuttleengine.ManifestItem`
- `plan:internal/shuttleparity#Unmet` -> `func Unmet(engine shuttleengine.Engine, spec shuttleengine.Spec) []shuttleengine.Requirement`
- `plan:internal/shuttleparity#PrepareEach` -> `func PrepareEach(engines shuttleengine.EngineSet, specs map[string]shuttleengine.Spec, stencilsDir, dir string, cfg shuttleengine.Config) (map[string]shuttleengine.Manifest, error)`
- `plan:internal/shuttleparity#Compare` -> `func Compare(manifests map[string]shuttleengine.Manifest) []shuttleengine.ManifestItem`
- `plan:internal/shuttleparity#SeatEnv` -> `type SeatEnv struct`
- `plan:internal/shuttleparity#Seat` -> `type Seat struct`
- `plan:internal/shuttleparity#Seats` -> `func Seats(env SeatEnv) ([]Seat, []error)`
- `plan:internal/shuttleparity#seatTables` -> `var seatTables []seatTable`
- `plan:internal/shuttleparity#seatTable` -> `type seatTable struct`
- `plan:internal/seatengine#Specs` -> `func Specs(geom Geometry, table Table) (map[string]shuttleengine.Spec, error)`
- `plan:internal/landingshed#FixReviewSeatTable` -> `func FixReviewSeatTable(stencilsDir string, languageSkills []string, cfg Config, reg modelspec.Registry) (seatengine.Table, error)`
- `plan:internal/burlerengine#Profile.SeatTables` -> `func (p *Profile) SeatTables(opts RunOpts, skills []string) []seatengine.Table`
- `plan:internal/seatengine#TestSpecs_MatchTheStartedSpecs` -> `func TestSpecs_MatchTheStartedSpecs(t *testing.T)`
- `plan:internal/shuttleparity#TestCompare` -> `func TestCompare(t *testing.T)`
- `plan:internal/shuttleparity#TestSeats_CodexManifestHoldsClaudes` -> `func TestSeats_CodexManifestHoldsClaudes(t *testing.T)`
- `plan:internal/shuttleparity#newParityFixture` -> `func newParityFixture(t *testing.T) parityFixture`
- `plan:internal/shuttleparity#parityFixture` -> `type parityFixture struct`

**Uses:**
- `internal/seatengine#seatSpec`
- `internal/seatengine#Prompts`
- `internal/seatengine/doc.go`
- `internal/landingshed#fixReviewTable`
- `internal/landingshed/doc.go`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine/doc.go`
- `internal/darnengine#DarnTable`
- `internal/landingshed#DescribeTable`
- `internal/landingshed#FixerTable`
- `internal/driftengine#ExpectTable`
- `internal/driftengine#JudgeTable`
- `internal/loomengine#PartsTable`
- `internal/loomengine#PlanTable`
- `internal/loomengine#PartsPlanTable`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#RaddleTable`
- `plan:internal/shuttleengines#NewWithSeams`
- `plan:internal/shuttleengine#MissingItems`
- `plan:internal/shuttleengine#ReadManifest`
- `plan:internal/shuttleengine#ErrHookUnrun`
- `internal/shuttleengine#InstallPassageReader`
- `internal/shuttleengine#RequirementSupporter`

**Intent:**
D1's per-seat comparison and the production code D10's verb shares with the test, in a new package `internal/shuttleparity` (it imports `shuttleengine`, `shuttleengines`, `seatengine`, `editdirective` and the seat-table builders; no concrete engine, per `PATTERN-shuttle-provider-seam`).

`doc.go` states the package: the shared probe and runner, the prompt items, the comparison, the seat catalog and its bounds.

Shared seams (D10):
- `ConfiguredSandbox` runs nothing and reports a sandbox that starts, so `codex.yaml`'s `auto` and `workspace-write` settle to `workspace-write` and `danger-full-access` probes nothing: the configured mode is what the manifest lists.
- `UnrunHooks` runs nothing and returns `shuttleengine.ErrHookUnrun`, so every plugin session-start context is listed by plugin and hook as omitted with that reason, and Codex logs no warning for it (card 4).

`PromptItems` returns the prompt-side items an engine cannot see, since an engine is told no stencils directory:
`edit-directive`, key `edit-directive`, delivered when the prompt holds the engine's copy from `editdirective.Directive(stencilsDir, engine)`, and no item otherwise.
`Unmet(engine, spec)` returns the requirements of `spec` the engine does not meet, through `RequirementSupporter`, with `RequirementForks` counted for a `ForkSubagents` spec as `Start` counts it; `Start` refuses such a spec on that engine.
`PrepareEach` runs, for each engine of the set, that engine's `Prepare` with its spec into `<dir>/<engine>`, reads the record back with `ReadManifest`, appends `PromptItems`, and returns the manifests by engine name.
A `Prepare` error is returned naming the engine.
An absent record is an error naming the engine, with the way forward: have that engine's `Prepare` write its manifest through `shuttleengine.WriteManifest`.
`Compare` returns `MissingItems(claude, codex)`, the Codex manifest checked against Claude's by key alone, so an omitted or unrun context counts as present.

The catalog: `seatTables` holds one `seatTable` per production table builder, each with `Builder`, the builder's glyph (such as `internal/darnengine#DarnTable`), `Holders`, the glyphs of every function whose body holds a `seatengine.Table` literal the builder returns (such as `internal/landingshed#describeSessionTable` for `DescribeTable`, `internal/landingshed#fixReviewTable` for `FixReviewSeatTable`, `internal/burlerengine#Profile.reviewTable` and `Profile.fixTable` for `Profile.SeatTables`), and `Build func(env SeatEnv) ([]seatengine.Table, error)`.
`Build` calls the builder with the current worktree's configuration and stand-ins only for a run's own inputs:
the told stencils and specs directories, the worktree's `*lymcwd.Location`, each builder's module config and the language skills loaded from `env.Dirs` through the loaders the wiring uses, and `env.Registry`;
a placeholder slug, branch names and paths under `env.Dir`, where `Build` first writes each input file a builder reads, such as a parts file with one part for `PartsPlanTable` and a decision record for `DescribeTable`.
The two unexported builders get exported entry points in their own packages: `landingshed.FixReviewSeatTable` fills `fixReviewInputs` with stand-in paths under the stencils directory and calls `fixReviewTable`; `(*Profile).SeatTables` returns `reviewTable` and `fixTable` with no marker values.
`SeatEnv` carries `StencilsDir`, `SpecsDir`, `Dir`, `Registry`, the `*lymcwd.Location`, `Dirs`, `Shortname`, `Slug` and `ParentName`.
`Seats(env)` builds every table and returns one `Seat` per seat of each table that built, each with `Name` (its strand role, `seatengine.SeatRole`), `Builder` and `Specs func(engine string) (shuttleengine.Spec, error)`, which sets every seat's `Engine` to engine and returns its spec from `seatengine.Specs` with the table geometry's `DefaultEngine` set to engine.
It returns beside them one error per builder that failed, naming the builder, so one failing builder costs only its own seats.
`seatengine.Specs` is `Prompts` plus `seatSpec` for each seat, the spec `startSeat` passes to `StartGated` short of its hold deferral, so the catalog launches no hand-built spec.

Tests, all tier0:
- `TestSpecs_MatchTheStartedSpecs` builds a fake table with `seatfake_test.go`'s fake shuttle and asserts each spec `Specs` returns equals the spec the fake records at start, after setting both specs' `HoldDeferral` to nil and asserting the recorded lead's was non-nil.
- `TestCompare` is a table over hand-built manifests: an item of a shared heddle target (`agents-prompt` passage, `agents-file` entry) absent from Codex is missing; an item keyed to `claude` (a `claude-file` entry) is not; an omitted plugin context in Codex counts present; an extra Codex item is no finding.
  Its `PromptItems` rows: a prompt holding the engine's edit-directive copy gives the item, and one without it gives none.
  Its `Unmet` rows: a `ForkSubagents` spec on an engine meeting no requirement gives `RequirementForks`, and a plain spec gives none.
- `TestSeats_CodexManifestHoldsClaudes`, `//lym:guard` since it builds the real tables over seeded stencils: `newParityFixture` seeds stencils with `stencilkit.Seed`, writes a fake checkout and Claude config directory (global CLAUDE.md, a plugin with one session-start hook, a skill) as `shuttleengines`' Codex launch fixture does, writes a marked `CLAUDE.local.md` and `AGENTS.override.md` holding its file entry's text, and installs a passage reader returning one `agents-prompt`, one `claude-prompt` and one `codex-prompt` passage and one `agents-file` entry.
  It fails on any builder error `Seats` returns.
  For every `Seat` whose Codex spec `Unmet` leaves empty, it runs `PrepareEach` on `shuttleengines.NewWithSeams` with `UnrunHooks` and `ConfiguredSandbox` and fails naming the seat and each item `Compare` returns, with the way forward: give the Codex engine a delivery for each missing item in `internal/shuttleengine/codexengine`.
  A seat whose Codex spec has unmet requirements is logged with them and not compared, since `Start` refuses it on Codex.
  It sets `CLAUDE_CONFIG_DIR` and `HOME` and installs the process passage reader, so it does not run in parallel.

Bound: the `MultiLLM` recipe row's table is built from its row's operator-written seat list, so it holds no seat in code; the catalog covers the tables built from code.

**Commit:** 6: Compare each seat's Codex manifest against Claude's
