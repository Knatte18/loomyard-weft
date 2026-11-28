# Card 6 — The parity package, its seat catalog and the per-seat parity test

**Create:**
- `internal/shuttleparity/doc.go`
- `internal/shuttleparity#UnrunHooks`
- `internal/shuttleparity#ConfiguredSandbox`
- `internal/shuttleparity#PromptItems`
- `internal/shuttleparity#Unmet`
- `internal/shuttleparity#PrepareEach`
- `internal/shuttleparity#Compare`
- `internal/shuttleparity#SeatEnv`
- `internal/shuttleparity#Seat`
- `internal/shuttleparity#seatsOf`
- `internal/shuttleparity#seatTables`
- `internal/shuttleparity#seatTable`
- `internal/seatengine#Specs`
- `internal/landingshed#FixReviewSeatTable`
- `internal/burlerengine#Engine.SeatTables`
- `internal/seatengine#TestSpecs_MatchTheStartedSpecs`
- `internal/shuttleparity#TestCompare`
- `internal/shuttleparity#TestSeats_CodexManifestHoldsClaudes`
- `internal/shuttleparity#newParityFixture`
- `internal/shuttleparity#parityFixture`
- `internal/shuttleparity#parityFixture.addSkills`
- `internal/shuttleparity#manifestEngine`
- `internal/shuttleparity#TestMain`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`

**Uses:**
- `internal/seatengine#seatSpec`
- `internal/seatengine#Prompts`
- `internal/seatengine/doc.go`
- `internal/landingshed#fixReviewTable`
- `internal/landingshed/doc.go`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine#Engine.probeTable`
- `internal/burlerengine#Engine.assistSeat`
- `internal/burlerengine#withAssistant`
- `internal/burlerengine#Engine.seatSkills`
- `internal/burlerengine#fixSeatValues`
- `internal/burlerengine#fixFilesIn`
- `internal/burlerengine#Engine.probeDirOf`
- `internal/burlerengine#New`
- `internal/lenses#Load`
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
- `internal/shuttleengines#NewWithSeams`
- `internal/shuttleengine#MissingItems`
- `internal/shuttleengine#ReadManifest`
- `internal/shuttleengine#ErrHookUnrun`
- `internal/shuttleengine#WebsterForkDenyKey`
- `internal/shuttleengine#InstallPassageReader`
- `internal/shuttleengine#RequirementSupporter`

**Intent:**
D1's per-seat comparison and the production code D10's verb shares with the test, in a new package `internal/shuttleparity` (it imports `shuttleengine`, `shuttleengines`, `seatengine`, `editdirective` and the seat-table builders; no concrete engine, per `PATTERN-shuttle-provider-seam`).

`doc.go` states the package: the shared probe and runner, the prompt items, the comparison, the seat catalog and its bounds.

The package declares the reference engine's name, `claude`, and the compared engine's name, `codex`, once each as exported named constants, and `Compare`, the manifest keys of `PrepareEach`, card 8's `Report`, every test of the package and card 8's `TestManifestCmd` in `internal/shuttlecli` use them.
The engines' own name constants are unexported, and an engine name is a registry key, which `PATTERN-shuttle-provider-seam` allows outside the engine directories.

Shared seams (D10):
- `ConfiguredSandbox` runs nothing and reports a sandbox that starts, so `codex.yaml`'s `auto` and `workspace-write` settle to `workspace-write` and `danger-full-access` probes nothing: the configured mode is what the manifest lists.
- `UnrunHooks` runs nothing and returns `shuttleengine.ErrHookUnrun` itself, never wrapped, so every plugin session-start context is listed by plugin and hook as omitted with that reason, and Codex logs no warning for it (card 4).
  Card 4's check matches the reason's text, which `pluginSessionStartContexts` copies from the runner's error, so a wrapped error would bring the warning back.

`PromptItems` returns the prompt-side items an engine cannot see, since an engine is told no stencils directory:
`edit-directive`, key `edit-directive`, delivered when the prompt holds the engine's copy from `editdirective.Directive(stencilsDir, engine)`, and no item otherwise.
`Unmet(engine, spec)` returns the requirements of `spec` the engine does not meet, through `RequirementSupporter`, with `RequirementForks` counted for a `ForkSubagents` spec as `Start` counts it; `Start` refuses such a spec on that engine.
Neither the parity tests nor `PrepareEach` skip a spec `Unmet` reports: `Prepare` refuses no requirement, so every seat is compared, per the decision record's `Added after Discussion (parent, 2026-11-17): Every seat is compared, including those Codex cannot start` entry.
`PrepareEach` first sets each spec's `Skills` with `cfg.Agents.Skills(spec.ConfigRole, spec.Skills)` when `ConfigRole` is non-empty, as `Runner.start` does before it calls `Prepare`, so the role's own skills reach both manifests.
An expansion error, such as a role `cfg.Agents` holds no skill list for, is returned naming the role, as `Runner.start` refuses it.
It then runs, for each engine of the set, that engine's `Prepare` with its spec into `<dir>/<engine>`, reads the record back with `ReadManifest`, appends `PromptItems`, and returns the manifests by engine name.
A `Prepare` error is returned naming the engine.
An absent record is an error naming the engine, with the way forward: have that engine's `Prepare` write its manifest through `shuttleengine.WriteManifest`.
`Compare(engines, manifests)` returns `MissingItems` of the reference engine's manifest and the compared engine's, the Codex manifest checked against Claude's by key alone, so an omitted or unrun context counts as present.
It drops the `guard` item keyed `shuttleengine.WebsterForkDenyKey` from that result when the set's compared engine does not answer `RequirementForks` through `shuttleengine.RequirementSupporter`.
The gate reads the engine's answer alone, not `Unmet` and no spec; the Claude manifest holds the item only for a `ForkSubagents` spec.
That item belongs to fork support, per the decision record's `Added after Discussion (parent, 2026-11-17): The webster fork deny is compared only on an engine that meets the fork requirement` entry.
Codex meets no requirement today, so the item is listed by Claude and not compared, and every other item of a fork seat still is.

The catalog: `seatTables` holds one `seatTable` per production table builder, each with `Builder`, the builder's glyph (such as `internal/darnengine#DarnTable`), `Holders`, the glyphs of every function whose body holds a `seatengine.Table` literal, or a `seatengine.Seat` literal that sets `Stencil`, the builder returns (such as `internal/landingshed#describeSessionTable` for `DescribeTable`, `internal/landingshed#fixReviewTable` for `FixReviewSeatTable`, `internal/burlerengine#Profile.reviewTable`, `Profile.fixTable` and `Engine.assistSeat` for `Engine.SeatTables`), and `Build func(env SeatEnv) ([]seatengine.Table, error)`.
`Build` calls the builder with the current worktree's configuration and stand-ins only for a run's own inputs:
the told stencils and specs directories, the worktree's `*lymcwd.Location`, each builder's module config and the language skills loaded from `env.Dirs` through the loaders the wiring uses, and `env.Registry`;
a placeholder slug, branch names and paths under `env.Dir`, where `Build` first writes each input file a builder reads, such as a parts file with one part for `PartsPlanTable` and a decision record for `DescribeTable`.
The two unexported builders get exported entry points in their own packages: `landingshed.FixReviewSeatTable` fills `fixReviewInputs` with stand-in paths under the stencils directory and calls `fixReviewTable`.
`(*Engine).SeatTables` returns `probeTable(p, opts)` and `p.fixTable(opts, fixSeatValues(p, fixFilesIn(e.probeDirOf())), e.seatSkills(p))`, the fix table as `ProbeFix` builds it.
The fixer stencil `burler-template-fixer` fills `review_path`, `review_format_path` and `instruction_fix_path` from the seat's values alone, and `seatengine.Specs` renders every seat through `Prompts`, which refuses an unfilled top-level marker, so a fix table without those values fails in `Specs`.
`probeTable` passes the seat from `assistSeat` through `withAssistant`, so when opts names an assistant the review table holds the assistant seat and the chair's assisted stencil, as an assisted round runs them.
The burler entry's `Build` makes the engine with `burlerengine.New` and no shuttle, since `SeatTables` runs no seat, over the worktree's `lenses.yaml` loaded with `lenses.Load`.
It calls `SeatTables` twice, each time with stand-in review and fixer-report paths under `env.Dir` and `RunOpts` whose `Assist` names a stand-in model, so the assistant seat, the one a Codex assist trial runs on Codex, is in the catalog:
once with a solo `Profile`, and once with a fanned one whose `ClusterFan`, `FixFan` and `opts.AssistFan` each name a fan that `lenses.yaml` defines.
The fanned call gives the forking reviewer and fixer and the fanned assistant stencil.
The two profiles carry different stand-in `RolePrefix` values, so no two seats of the catalog share a name.
`SeatEnv` carries `StencilsDir`, `SpecsDir`, `Dir`, `Registry`, the `*lymcwd.Location`, `Dirs`, `Shortname`, `Slug` and `ParentName`.
`seatsOf(env, tables)` takes the catalog as an argument, so `TestCompare` and card 8's `TestReport` run it on fixture tables and `TestSeats_CodexManifestHoldsClaudes` on `seatTables`.
It builds every table and returns one `Seat` per seat of each table that built, each with `Name` (its strand role, `seatengine.SeatRole`), `Builder` and `Specs func(engine string) (shuttleengine.Spec, error)`, which sets every seat's `ModelChoice` to `ModelChoice{Engine: engine}` alone and returns its spec from `seatengine.Specs` with the table geometry's `DefaultEngine` set to engine.
The seat's `Model`, `Effort` and `Version` are cleared, since the manifest compares what the session is given and not which model runs it, and each engine's `Prepare` validates its own model vocabulary: a Claude effort or version pin would make Codex's `Prepare` refuse the spec.
It returns beside them one error per builder that failed, so one failing builder costs only its own seats.
Each error names the builder's glyph and wraps its error, with the way forward: fix the stand-in inputs that builder's `seatTables` entry's `Build` writes in `internal/shuttleparity`, since the builder ran on those stand-ins and not on a run's inputs.
`seatengine.Specs` is `Prompts` plus `seatSpec` for each seat, the spec `startSeat` passes to `StartGated` short of its hold deferral, so the catalog launches no hand-built spec.

`testOnlyCodeAllowed` in `cmd/lym/testonlycode_linux_test.go` gains the entries `internal/seatengine.Specs`, `internal/landingshed.FixReviewSeatTable` and `internal/burlerengine.Engine.SeatTables`, each `Why` saying it has no caller until `lym shuttle manifest` lands and that card removes the entry.
Only `internal/shuttleparity` calls them, and no main package imports it before card 8, so without the entries a batch gate ending before card 8 fails `PATTERN-test-only-code`'s guard.
The functions of `internal/shuttleparity` itself need no entry, since no main or test-support package imports it before card 8.

The package's `testmain_test.go` holds `TestMain`, mirroring `internal/seatengine/testmain_test.go`: it calls `gitkit.HermeticGitEnv()` and then `os.Exit(tmuxkit.Main(m))`, so the package's tests run under `PATTERN-test-isolation`'s `tmuxkit.Main`.

Tests, all tier0:
- `TestSpecs_MatchTheStartedSpecs` builds a fake table with `seatfake_test.go`'s fake shuttle and asserts each spec `Specs` returns equals the spec the fake records at start, after setting both specs' `HoldDeferral` to nil and asserting the recorded lead's was non-nil.
- `TestCompare` is a table over hand-built manifests: an item of a shared heddle target (`agents-prompt` passage, `agents-file` entry) absent from Codex is missing; an item keyed to `claude` (a `claude-file` entry) is not; an omitted plugin context in Codex counts present; an extra Codex item is no finding.
  Its fork-deny rows run over a set whose `codex` engine is a `manifestEngine` fake, with a Claude manifest holding `guard webster-fork-deny` and a Codex one lacking it:
  a fake meeting no requirement gives no finding, and a fake meeting `RequirementForks` gives exactly that item, the tripwire that fails the parity test the day Codex reports fork support until it delivers a fork deny.
  Its `PromptItems` rows: a prompt holding the engine's edit-directive copy gives the item, and one without it gives none.
  Its `Unmet` rows: a `ForkSubagents` spec on an engine meeting no requirement gives `RequirementForks`, and a plain spec gives none.
  Its `UnrunHooks` row: the error returned equals `shuttleengine.ErrHookUnrun` under `==`, so a wrapped error fails it.
  Its `PrepareEach` rows run over two `manifestEngine` fakes, a fake engine whose `Prepare` writes a given manifest, fails or writes none:
  a failing `Prepare` returns an error naming its engine, and one that writes no record returns the error naming its engine and `shuttleengine.WriteManifest`;
  a spec with a `ConfigRole` reaches each fake's `Prepare` with the role's skills from `cfg.Agents` followed by its own, and a role `cfg.Agents` holds no list for returns the error naming the role.
  Its `seatsOf` row runs two fixture `seatTable` entries, the second's `Build` failing: the first entry's seats still build, and the one error returned names the second's glyph and wraps its error.
  A further `seatsOf` row runs a fixture table whose seat `ModelChoice` carries `Model`, `Effort` and `Version`: for each engine, the seat's `Specs` gives a spec whose `ModelChoice` is that engine alone.
- `TestSeats_CodexManifestHoldsClaudes`, `//lym:guard` since it builds the real tables over seeded stencils: `newParityFixture` seeds stencils with `stencilkit.Seed`, writes a fake checkout and Claude config directory (global CLAUDE.md, a plugin with one session-start hook) as `shuttleengines`' Codex launch fixture does, writes a marked `CLAUDE.local.md` and `AGENTS.override.md` holding its file entry's text, and installs a passage reader returning one `agents-prompt`, one `claude-prompt` and one `codex-prompt` passage and one `agents-file` entry.
  The fixture's worktree codex.yaml layer sets `sandbox_mode: auto` and `skill_roots` to a directory of the fixture, and every compared seat's Codex manifest lists `workspace-write` in its sandbox `settings` item, the mode `ConfiguredSandbox` settles to.
  The `cfg` it hands `PrepareEach` is loaded with `shuttleengine.LoadConfig` over the fixture's config dirs, so `cfg.Agents` is the hub's `agents.yaml` and holds a skill list for every seat role.
  It also writes a hub `language.yaml` layer listing at least one `<plugin>:<name>` skill, since seat and producer specs name skills only through the language skills and `ConfigRole`, so an empty list would leave the skill channels unexercised.
  It calls `seatsOf(env, seatTables)` and fails on each builder error it returns, printing the error with its way forward.
  It then collects every skill name the built specs hold after the expansion `PrepareEach` applies, `cfg.Agents.Skills(spec.ConfigRole, spec.Skills)` for a spec with a `ConfigRole`, so the role's own skills are compared too, and calls `fx.addSkills` with them before any `PrepareEach` runs.
  `addSkills` writes each name `<plugin>:<name>` as `<skill_roots dir>/<marketplace>/<plugin>/<version>/skills/<name>/SKILL.md`, the plugin-cache layout `codexengine`'s `skillPath` resolves.
  For Claude it writes each as `<install dir>/skills/<name>/SKILL.md` of a plugin `<plugin>` the fixture's Claude config directory lists as installed, the directory `claudeengine`'s `skillListing` scans and lists as `<plugin>:<name>`, so a seat's skills resolve for both engines and no `skill-load` or `skill-listing` item is missing by construction.
  A name without a colon is a fixture error, since Codex lists no `skill-load` item for it whatever file exists.
  For every `Seat`, a seat Codex cannot start included, it runs `PrepareEach` on `shuttleengines.NewWithSeams` with `UnrunHooks` and `ConfiguredSandbox` and fails naming the seat and each item `Compare` returns, with the way forward: give the Codex engine a delivery for each missing item in `internal/shuttleengine/codexengine`.
  It sets `CLAUDE_CONFIG_DIR` and `HOME` and installs the process passage reader, so it does not run in parallel.

Bound: the `MultiLLM` recipe row's table is built from its row's operator-written seat list, so it holds no seat in code; the catalog covers the tables built from code.

**ImpactSummary:** `testOnlyCodeAllowed` gains interim entries for the production functions no main package reaches until card 8, and card 8 removes them; no existing function changes.

**Commit:** 6: Compare each seat's Codex manifest against Claude's
