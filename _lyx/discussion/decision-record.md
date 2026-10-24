# Decision record — dotnet-scoped-verify

## Goal

Verify gates in a .NET hub build once and test only what a card, batch or diff touches, with the whole solution kept to the landing gates.
The same task delivers the remaining wants of the Models verification protocol: fewer full-suite runs (Wanted 4), an exclusive slot hold for timing runs (Wanted 5), one build shared by several tiers (Wanted 6), and a way for a hub to carry its repo rules into code review.

## Scope

In scope:

- A third `gate.yaml` scoper, `dotnet`, beside `go` and `none`, mapping changed C# code to the test projects and test classes that exercise it, and running only those through `dotnet test --filter`.
- A `{filter}` tier placeholder and a tier `filter` key, so a tier's own filter (such as `Category!=Slow`) and the scoper's class filter combine into one `--filter` argument.
- A hub-level `build` step in `gate.yaml` that a `lyx gate test` call runs once before the tiers that opt in with `shared_build: true`, whose commands then run `--no-build`.
- A hub-level `inert_paths` list in `gate.yaml`: a diff that touches only inert paths runs no test tier but a `carry: false` one, at narrowed and whole gates alike.
- A tier `carry: false` key: such a tier is never carried and runs whole in a `lyx gate test` call that lists it, unless the call names explicit targets or another listed tier covers it (Decision 8).
- Documentation: the package docs of every package touched, `docs/overview.md`, and a new `docs/dotnet-hub.md` that sets up a .NET hub with the Models repository as its worked example: `scoper: dotnet`, tiers with `{targets}`, `{filter}`, `filter` and `shared_build`, the `build` step, `inert_paths`, the recommended gate-key scopes (Decision 10), the timing tier and where a `carry: false` tier stays whole (Decision 8), `refuse_raw: ["dotnet test"]`, the zero-test exit option for a Microsoft.Testing.Platform hub (Decision 4), the warm-process behaviour per platform and the MSBuild escape hatches to leave unset (Decision 7), the repo-rules lens (Decision 9), and taking a `lyx gate report` baseline on OSL-1046 before installing the binary this task lands.

Out of scope:

- The numerical baseline (Required 2 and its detached capture), which belongs to note `regression-baseline-verify`; that note's capture and compare tiers rely on `carry: false` (Decision 8).
- Required 1 and Required 3, which the board entry assigns to the landed task `test-tiers`.
- Measuring build and test minutes on OSL-1046: Models is never cloned outside OSL-1046, so the operator measures there with `lyx gate report`, and the first real C# run goes through the `cs` sandbox hub of note `csharp-sandbox-hub`.
- Writing any hub's own config: the Models hub and the `cs` hub set their `gate.yaml` and `lenses.yaml` from `docs/dotnet-hub.md`.

## Decisions

### 1. A `dotnet` scoper in a new package

`gate.yaml`'s `scoper` accepts `dotnet`.
Its derivation lives in a new package, `internal/dotnetimpact`, the sibling of `internal/impactset`, told the worktree, the tier's `dir` and either a base commit or the cards' seeds, and resolving no cwd.
`gatecli.computeScope` calls it for the computed, `--base` and `--card` forms under the `dotnet` scoper, once per distinct base and tier `dir`, as it calls `impactset.DeriveTargets` per base under `go`.
Rationale: the scoper seam already exists, and C# needs a map Go's import graph cannot give, since C# tests sit in separate, often shared, test projects.

### 2. The map: a type-name closure over the C# sources, selecting test classes

The derivation reads every `.cs` file under the tier's `dir`, skipping `bin/`, `obj/`, `.scratch/` and the lyx and VCS directories, with a C# lexer that drops comments and yields identifier tokens and type declarations.
The lexer drops the literal text of a string but keeps the code inside an interpolated string's holes, regular and raw, as identifier tokens, so `$"{Formatter.Format(v)}"` names `Formatter`.

- Diff seed: the types declared in each changed `.cs` file, in its version at the base and in the working tree, so a renamed or deleted type still seeds; an untracked `.cs` file counts as changed, read as `impactset` reads untracked files.
- Card seed: for each Target, the file a self glyph names, the type names a member glyph or `plan:` handle spells in its owner chain (`NORCE.Models#Foo.Bar` seeds `Foo`), or, for a glyph whose owner chain is empty, its own name as a type, since C# has no namespace-level members (`NORCE.Models#Circle` seeds `Circle`); each seeded name has its generic or arity suffix stripped (`Box<T>` seeds `Box`); the seeds come from a new exported function of `internal/planparser`.
- The closure adds, until nothing changes, every type declared in a file whose identifiers name a type already in the set.
- A test class is a class carrying a test attribute (`TestFixture`, `TestClass`) or holding a method that carries one (`Test`, `TestCase`, `TestCaseSource`, `Theory`, `Fact`, `TestMethod`, `DataTestMethod`), each matched in its plain, `Attribute`-suffixed and namespace-qualified spelling and inside a multi-attribute list; its test project is the `.csproj` in the nearest ancestor directory.
- A non-abstract class whose declared base class, followed transitively through the scanned sources, is a test class is itself a test class with its own filter term, since NUnit names an inherited test after the derived class.
- A test class is selected when its file changed, its file is a card Target, or its file names a type in the closure; a selected abstract test class selects every class deriving from it, and an abstract class gets no filter term of its own.
- The result is, per test project holding a selected class, the project path relative to the tier's `dir` and one filter term per selected non-abstract class that matches every test of that class, over-matching allowed (a class name that prefixes another selects both), never under-matching.
  A term is `FullyQualifiedName~` followed by the class's namespace and class chain, a nested class joined to its outer class by `+` as VSTest reports it (`FullyQualifiedName~Ns.Outer+Inner`).
  It also reports which empty scope it is: a non-empty seed that selected no test class, apart from an empty or inert-only diff or a card whose Targets are all inert, so the call can tell the two apart (Decisions 3 and 6).

Bound: a name match over-selects safely, and the closure misses only references that never spell the type's name: dispatch through a base class or interface, such as a test that names only `Shape` or `IFoo` while the change is in `Circle` or a DI-resolved implementation, extension-method calls, `var`-typed results of a factory, reflection and string-named lookups, and source-generated code.
What it misses is caught by the gates that run whole (Decision 10).
Rationale: the name closure narrows inside a shared test project with no coverage run and no stored map, and its misses are bounded by the whole gates exactly as `impactset`'s are.

### 3. When the `dotnet` scoper falls back to whole

The derivation returns whole, with the reason printed on stderr as the `go` scoper's fallback is, when any of these holds:

- the base is empty or not an ancestor of HEAD;
- a changed path, by its new or its old name, is neither a `.cs` file nor an inert path (Decision 5), so a `.csproj`, `.props`, `.targets`, `.sln`, `global.json`, `nuget.config`, `.resx` or test-data change runs whole;
- a changed `.cs` file lies outside the tier's `dir`, or under it with no `.csproj` in any ancestor directory up to the `dir`;
- a changed `.cs` file declares no type in either version, such as `GlobalUsings.cs`, `AssemblyInfo.cs` or a top-level-statement file;
- under `--card`, a Target's file, under the path rules the diff forms apply, is a non-`.cs` path that is not inert, or the Target yields neither an existing `.cs` file nor a type name, such as a deleted file or a namespace-only self glyph; a Target whose file is inert (Decision 5) is dropped from the seed;
- a selected test class's file has no `.csproj` in any ancestor directory up to the tier's `dir`, or its nearest ancestor directory holding a `.csproj` holds more than one;
- a git read fails or a source file under the `dir` cannot be read.

An empty diff yields the empty scope and runs no build, as an empty diff does under `go`, unless a listed `carry: false` tier opts in with `shared_build`: that tier still runs whole (Decision 8), with the build before it (Decision 6).
A diff whose changed paths are all inert, or a card whose Targets are all inert, yields the empty scope, so no listed tier's command runs but a `carry: false` tier's (Decision 5).
A closure that selects no test class from a non-empty seed yields the empty test scope.
The call then decides per listed tier, whether or not another listed tier opts in: a tier that opts in with `shared_build` runs no test command, the build step running once inside the first such tier's slots (Decision 6), and a tier that does not runs whole, with the reason printed on stderr like the fallbacks above, so a compile break fails the card gate either way.
Rationale: every case the derivation can detect as beyond the name closure's sight runs whole; the misses Decision 2's Bound lists are left to the whole gates of Decision 10.

### 4. `{filter}` placeholder and tier `filter` key

A tier may hold a `filter` string and the placeholder `{filter}`, under any scoper.
`{filter}` renders, quoted for the platform shell by `verifyrun.QuoteArg`, as `--filter <expr>`:

The rendering is keyed on each invocation of the tier's own share of the call's scope, never the call's scope as a whole:

- an invocation the `dotnet` scoper did not narrow to a selected project with its terms, including a tier running whole inside a narrowed `dotnet` call (a `carry: false` tier, a whole-fallback base group or a command lacking `{targets}`), a project past `greenCommandBudget`, explicit targets, darn's tree-scope targets, a narrowed or `--card` `go` scope and the guard step: `<expr>` is the tier's `filter`, and `{filter}` renders empty when the tier has none;
- an invocation the `dotnet` scoper narrowed to a selected project with its terms: `<expr>` is `(<tier filter>)&(<term>|<term>...)`, or the terms alone when the tier has no `filter`.

Under a narrowed `dotnet` scope a tier whose command holds `{targets}` runs once per selected test project, in the tier's `dir`, with that project as `{targets}` and its terms in `{filter}`, under one slot hold and one `timeout_min` covering every invocation, as a narrowed `go` scope runs once per module.
A tier whose command lacks `{targets}` runs once, whole, with its tier filter only, as such a command does under every narrower scope today.
A project whose rendered filter would exceed `greenCommandBudget` (the parent's decision added after Discussion) falls back to that project's whole invocation, with the tier's own filter only, still narrowed to that project.
A tier command without `{filter}` under the `dotnet` scoper runs each selected project with no class filter.

A narrowed run may name a project the tier does not run as a test project, such as a Models gated project tier0 runs without `-p:RunSlowTests=true`, so it evaluates with `IsTestProject=false`.
Under VSTest that run tests nothing and succeeds: the test platform's `VSTest` target runs its `VSTestTask` and its build call only when `'$(IsTestProject)' == 'true'` (microsoft/vstest `src/Microsoft.TestPlatform.Build/Microsoft.TestPlatform.targets` at `v17.12.0`, the test platform of the .NET 9 SDK Models builds with).
A run whose composed filter matches no test in a project the tier runs also arises, when the tier's `filter` excludes every test of a selected class, such as `Category!=Slow` over a class whose tests are all `Slow`.
VSTest warns and exits 0 there, since `TreatNoTestsAsError` defaults to false (Microsoft Learn, "Configure unit tests by using a .runsettings file").
Microsoft.Testing.Platform exits 8 for a session that ran zero tests, and `--ignore-exit-code 8`, or `TESTINGPLATFORM_EXITCODE_IGNORE=8` in the environment, turns that code into 0 (Microsoft Learn, "Microsoft.Testing.Platform exit codes").
`docs/dotnet-hub.md` therefore tells a hub on Microsoft.Testing.Platform to put `--ignore-exit-code 8` in the command, or `TESTINGPLATFORM_EXITCODE_IGNORE: "8"` in the `env`, of every tier whose command holds `{filter}`.
`gateslot.LoadConfig` refuses, naming the tier and `lyx config gate`, a `filter` on a tier whose command lacks `{filter}`.
Bound: a tier's `filter` applies at whole scope too, landing gates included, so a filter that matches no test passes the tier vacuously and writes a whole green entry later gates carry.
It is bounded by sitting inside the executable, hub-file-only `tiers` key, whose command can already run anything; `docs/dotnet-hub.md` tells a hub that a landing tier's `filter` only excludes categories.
Rationale: `dotnet test` takes one `--filter`, and VSTest's filter grammar combines conditions with `&`, `|` and parentheses (Microsoft Learn, "Run selected unit tests", `dotnet test --filter`), so lyx composes the tier's filter and the scope's into one expression.

### 5. `inert_paths`: a doc-only diff runs no tier

`gate.yaml` gains `inert_paths`, a list of glob patterns, empty in the template, a guard key held in the hub file only.
Patterns are matched against paths relative to the worktree root (the git root), never the tier's `dir`: a pattern without `/` matches a changed path's base name and a pattern with `/` matches the worktree-relative path, both through `path.Match`.
The rules below apply only when `inert_paths` is non-empty, and only to a diff holding at least one changed path, so a hub with the empty list behaves exactly as today.

- Under every scoper, a computed or `--base` scope whose changed paths are all inert is the empty scope, so no listed tier's command runs but a `carry: false` tier's, which still runs whole (Decision 8); the `none` scoper therefore reads the diff from the tier's base for this check, and otherwise stays whole.
  The changed paths include untracked files, read through `git ls-files --others --exclude-standard` as `impactset` reads them, so an untracked non-inert file keeps the scope whole.
  Under the `go` scoper the empty scope keeps planning the guard step, as a Markdown-only empty scope does today.
- The `dotnet` scoper drops inert paths from its changed set (Decision 3).
- At a whole-scope gate, a tier with no green entry for the current tree is carried when the green record holds a whole entry for the tier, under its current fingerprint, empty args and the same module directory, whose commit is an ancestor of HEAD and whose diff to the clean working tree touches only inert paths; the carry writes a whole entry for the current tree.
- A non-empty `inert_paths` list joins every tier's fingerprint, so a change to the list voids every green entry written under the old list, carried ones included.

Bound: a test that reads an inert file is skipped when only that file changes; the hub chooses its patterns knowing this, and loomyard, whose tests read Markdown, keeps the empty list.
The carry obeys the green record's existing rules: a dirty worktree reads no record, a tier with `carry: false` never carries, and the newest-20 whole entries per fingerprint bound how far back an ancestor is found.
Rationale: Wanted 4's doc-only rule, generalised to a hub-declared list so no language or file kind is hard-coded, and placed in the green record so a tier is still skipped only on `lyx gate test`'s own record (`PATTERN-gate-slot`).

### 6. A shared build step that also restores

`gate.yaml` gains a `build` mapping, an executable key held in the hub file only, seeded in the template as an empty mapping that means no build step: `command`, `dir` (default `.`), `env`, `timeout_min` (at least 1) and `about`.
Its command takes no placeholder, so the words after `--` never reach it.
A tier opts in with `shared_build: true`, and its command then carries `--no-build` by the hub's choice.

- A `lyx gate test` call runs the build command once, inside the slot hold of the first opted-in tier in plan order whose command will run or whose `dotnet` scope is an empty test scope from a non-empty seed (Decision 3), before that tier's command.
  Such an empty-test-scope tier acquires its tier's slots to run the build, though an empty scope takes no slot today, and runs no test command inside them.
- The build is bounded by its own `timeout_min`, counted from the hold; the tier's own `timeout_min` starts after the build ends.
- A call whose opted-in tiers are all carried or have an inert-only, all-inert-card or empty-diff empty scope runs no build.
- The build always builds the hub's whole build target as its command names it (for Models the solution, with `-p:RunSlowTests=true -p:RunBaseline=true`), whatever the tiers' scopes, and relies on MSBuild's incremental up-to-date checks for untouched projects.
- The build's output passes through to the call's stdout and stderr and its log as a tier's does, so a verify log holds the compile error.
- A failed or timed-out build is a failed run, counted in plan order before the opted-in tier it precedes, and the call exits with the first failed run's code, so with the build's code, or 1 for a timeout, when no earlier run failed; every opted-in tier left in the call fails without running its command, each with a stderr line and run record naming the build failure and its log; tiers not opted in still run, and the closing line names `build` as a failed run.
- The build prints its own summary line, distinct from the `tier<N>` summary line, and the opted-in tiers it blocks print no failed `tier<N>` summary line.
  `gateslot` reads the build summary line beside `FailedTier`, and a log whose first failing summary line is the build's gives Merge-In's and Finalize's landing failure record `build: true`, with `tier` the first opted-in tier the build blocked and failing tests empty, so the landing fixer is pointed at a compile break, not at tests.
- A timed-out build prints a build timeout line naming `lyx config gate --set 'build={...}'` as the way forward, and `verifytree` reads it as timed out as it reads the tier timeout line.
- The build writes its own run record and log through `gateslot.WriteRunRecord`, named with `build` in place of `tier<N>` and carrying the scope form `build`; `lyx gate report` totals build runs and their time apart from tier runs.
- An opted-in tier's fingerprint also hashes the `build` declaration, so a changed build command never carries a tier.
- `gateslot.LoadConfig` refuses, naming `lyx config gate`: `shared_build: true` with an empty `build`, a non-empty `build` without `command` or `timeout_min`, and a `timeout_min` below 1.

Restore: the want "restore once per run, then build and test with `--no-restore`" is met without a lyx restore step.
The hub's build command restores implicitly, and the opted-in tiers' `--no-build` runs never restore.
NuGet's no-op restore compares its inputs against the `project.nuget.cache` hash and does nothing when they are unchanged, so restore does real work only when a project file or package reference changed, and a cleaned `obj/` still restores.
Bound: a tier with `shared_build: true` whose command lacks `--no-build` builds again inside its own run; lyx does not inspect the command.
Rationale: Wanted 6 saves a build per extra tier, and a build before narrowed runs saves one per selected test project.

### 7. Warm build processes

lyx sets nothing that disables MSBuild node reuse or the shared compiler server: no `MSBUILDDISABLENODEREUSE`, no `-nodeReuse:false`, no `UseSharedCompilation=false` in any tier, build or holder environment.
A hub that wants the opt-in MSBuild server sets `DOTNET_CLI_USE_MSBUILD_SERVER` in its `build` and tier `env`.

- Linux: idle nodes outlive the tier's tied child, since its process group is killed only on cancel and `runproc.End` reaches only processes still below a record, so they serve later gate calls until the SDK's idle timeout.
- Windows: the tied child sits in a kill-on-close job object, which every process it starts inherits, so nodes and the compiler server end with each build or tier run; lyx adds no job breakaway, because a job permitting breakaway would let test processes escape the tie too.
  On Windows the saving is the one build per call of Decision 6 and the `--no-build` test runs, not cross-call warmth.
- Output: a warm node holds none of the tier child's output pipe, so the child's `Wait` returns when the build or test process itself exits and lyx adds no wait bound for lingering output holders.
  MSBuild's node launcher starts a node with stdin, stdout and stderr redirected on Unix and, on Windows, with `INVALID_HANDLE_VALUE` standard handles under `STARTF_USESTDHANDLES` and `bInheritHandles: false` (dotnet/msbuild `src/Build/BackEnd/Components/Communications/NodeLauncher.cs` at `vs17.12`, the MSBuild of the .NET 9 SDK).
  The Roslyn compiler server is started the same way on both platforms (dotnet/roslyn `src/Compilers/Shared/BuildServerConnection.cs`, `TryCreateServer`, at `release/dev17.12`).

Bound: MSBuild's main-branch launcher, newer than the .NET 9 SDK's, leaves a Unix node's stdio unredirected when `MSBUILDENSURESTDOUTFORTASKPROCESSES=1` or `MSBUILDNODEWINDOW` is set, so on a later SDK a hub that sets either in a `build` or tier `env` gets a gate call that waits on idle nodes until `timeout_min`; `docs/dotnet-hub.md` says to set neither.

Rationale: node reuse is the SDK default for multi-node builds (MSBuild team, "Node reuse in multiproc MSBuild"), so keeping processes warm means not defeating it where the process tree allows it, and the tie that ends a tier's tree on Windows is worth more than node warmth.

### 8. `carry: false`: a tier that runs whole wherever it is listed

A tier may set `carry: false`.
Inside one `lyx gate test` call that lists it, the green record never carries it and it writes no green entry, and every scope form but explicit targets runs it whole unless another listed tier covers it, so neither a computed scope nor an inert-only or no-class diff can leave it unrun.
`carry`, `filter` and `shared_build` join the tier fingerprint only when set to a non-default value, as a non-empty `inert_paths` does (Decision 5), so an existing tier's fingerprint and green entries are unchanged.

Wanted 5's exclusive lock is a tier the hub declares with `slots: all`, `manual: true` and `carry: false`, whose command runs the timing work, such as a BenchmarkDotNet project, with the agent's words after `--` in `{args}`.
`slots: all` takes the pool's drain, waits for every running holder to finish and blocks every new gate run of the hub until it ends, so other worktrees' gate runs wait while their agents keep working, and correctness runs keep sharing the pool's slots among themselves.
The timing call passes `--wait`, so its drain is not dropped at the keep-cache-warm cap and re-queued behind new single-slot runs.
Note `regression-baseline-verify`'s capture and compare tiers set `carry: false` for the same reason: inside a `lyx gate test` call that lists them, a capture on an unchanged tree must write its baseline, and a compare must never be narrowed or carried; the sites the next Bound names stay that note's to avoid.
Bound: three paths still skip or narrow a `carry: false` tier.
A verify site's verified-tree record skips a gate call on a tree the same command already passed (`verifytree.Verify`); explicit targets, the calls of darn's tree-scoped sites included, narrow it or leave it unrun; and a listed tier whose `covers` names it drops it from the call.
`docs/dotnet-hub.md` therefore tells a hub to keep a `carry: false` tier out of darn's gate keys and every tier's `covers`, and to run a tier that must run on an unchanged tree, such as a timing run or a capture, only through a direct `lyx gate test` call, never a verify site.
Bound: a raw build an agent runs outside `lyx gate test` is not held, as Wanted 5 accepts; refusing raw `dotnet test` through `refuse_raw` is the hub's choice.
Rationale: the pool's drain is already the hub-wide exclusive lock Wanted 5 asks for; only a carried or narrowed timing tier could have skipped a measurement.

### 9. Repo rules through a hub lens, documented

The Models review rules are lens prose in the hub's own `lenses.yaml`, added to the fan the Code-Review perch reads; lyx gains no rule engine.
`docs/dotnet-hub.md` carries the lens as part of its worked Models example: no test in the three test projects runs, mirrors or references a `Program*` demonstration (checked by grepping the test projects for `Program0NN_`); comparisons against stored output live in `NORCE.Models.NumericalBaseline` alone, while an in-process test may byte-compare two arms it builds itself; no argument guards and no try/catch around the repo's own code.
Rationale: the board entry calls these "not tool features, but things a configurable review checklist should carry", and `lenses.yaml` is that checklist.

### 10. Gate keys for a .NET hub: the whole solution at landing only

`docs/dotnet-hub.md` recommends, for a .NET hub: `card` and `batch` at scope `card`, `webster` at scope `diff`, `webster-fix`, `darn-write` and `code-burler-fix` at scope `diff`, and `merge-in` and `finalize` at scope `all`.
Merge-In is then the one planned whole-solution run per task, besides the card, batch and webster gates Decision 3 sends whole; Finalize carries it from the green record when no commit landed between, and runs whole again only after a Code-Review fix or a moved parent changed the tree.
The template's gate keys and loomyard's own keep their scopes.
Rationale: the board wants the whole solution once, at the final gate; Merge-In is the first gate over the merged parent, so a run there cannot be saved, and a whole Webster end gate before it would be the second.

### Added after Discussion (parent, 2026-10-24): The over-long filter bound reuses greenCommandBudget

Decision: The gate's bound on a rendered filter is `greenCommandBudget` in `internal/gatecli/green.go` (6000 bytes of rendered targets), reused, not a new constant: a filter that renders past it falls back to the whole invocation, as a carried-over remainder already does. If the bound must be read outside `gatecli`, move the constant to where both readers see it rather than copying the number.

Rationale: F12: one bound for one hazard, the shell refusing an overlong command line; 6000 bytes stays under cmd's 8191-character limit with room for the command around the targets.

## Constraints

- `PATTERN-gate-slot`: the build step runs inside held slots of a tier run; a tier is still skipped only on `lyx gate test`'s green record and only over a clean tree, the inert carry included.
- Green record under the `dotnet` scoper: only whole entries carry, as under `none`; an invocation that ran with scope filter terms, or over fewer test projects than the whole, writes no green entry, even when its `{targets}` equal the tier's `all_targets`, so a filtered pass never stands in for a whole run.
  A whole pass of every listed tier, the slow tier included, carries at a later gate on the same tree; this task adds no record for it.
- `PATTERN-config-layers`: `build`, `inert_paths` and the new tier keys load through `gateslot.LoadConfig` over `configengine`; the template holds `build: {}` and `inert_paths: []`, and `build` joins `gateslot.ConfigOpenMaps` beside `tiers` and `tier_parallel`, so `configengine` carries its sub-keys whole through the layer stack and the reconcile after a binary change never prunes a hub's values, while `gateslot.LoadConfig`'s refusals in Decision 6 validate its keys; `build` is executable and `inert_paths` a guard key, both hub-file only.
- `PATTERN-cwd-resolution`: `internal/dotnetimpact` is told absolute paths and never imports `internal/lyxcwd`.
- `PATTERN-gitrepo-client-boundary`: the derivation's diff and base reads go through `gitexec.Run` or `gitrepo`, as `impactset`'s do.
- `PATTERN-spawn-observability`: the build step spawns through the tier runner's tied child and logged spawn; the derivation spawns nothing beyond git.
- `PATTERN-tolerant-records`: the build step's run record and the carried entries go through the existing tolerant writers; no new record file is added.
- `PATTERN-sole-parsers`: card seeds come from `internal/planparser`, never a second plan parse in `gatecli` or `dotnetimpact`.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: the derivation's tests are `tier0` over fixture trees written in the test; a test that spawns `dotnet` is `tier1` and skips when `dotnet` is absent or has no SDK, as `TestCSharpFixtures_CompileWithDotnet` does, and a test that restores packages restores offline and skips when that restore finds none.
- No PATTERN entry changes: every new rule is local to the gate packages; the `PATTERN-gate-slot` background's "Every gate build keeps `-trimpath`" line is Go-specific and is narrowed to Go builds in the same card that adds the build step.

## Auto-mode assumptions

- A test project is the `.csproj` in the nearest ancestor directory of a test class's file.
- Naming Models' project names and repo rules in loomyard's public docs is acceptable, since issue #609 already names them.

## Open risks

- The name closure on a large codebase may select most test classes for a change to a widely named type; a closure selecting classes in every test project runs one `dotnet test` per project in sequence inside the tier, which can cost more than one solution-level run.
- The build step's whole-solution incremental build may cost more than building one test project on a solution with many large projects untouched by the change; Models' observed full build is ~30 s.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - build-not-an-open-config-map (class design, severity MEDIUM, open in rounds 2)
  - card-seed-misses-type-glyphs-and-inert-targets (class design, severity MEDIUM, open in rounds 2)
  - filter-term-spelling-unstated (class design, severity MEDIUM, open in rounds 2)
  - lexer-interpolation-holes-break-bound (class design, severity MEDIUM, open in rounds 2)
  - tier-filter-key-has-no-bound (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Under `scoper: dotnet`, a diff changing one production `.cs` file selects the test classes whose files name a type it declares or a type that transitively names one, and runs each owning test project once with `{targets}` the project and `{filter}` the composed expression.
- A test class whose own file changed is selected even when it names no changed type.
- A renamed or deleted type, read from the base version of its file, still seeds the closure under a diff form.
- A card whose Target is a member glyph seeds its owner type and selects that type's test classes; a card whose Target is a type glyph such as `NORCE.Models#Circle` seeds that type; a card Target whose file is inert is dropped from the seed, and a card whose Targets are all inert runs no tier but a `carry: false` one and no build; a card whose Target yields neither a file nor a type name, or names a non-`.cs` file that is not inert, runs whole.
- Test classes are found under each attribute spelling: plain, `Attribute`-suffixed, namespace-qualified and inside an attribute list; a nested test class is found and its term joins it to its outer class with `+`, so its tests run.
- A non-abstract fixture whose tests live only in an abstract base class is a test class; a change that selects the base selects the derived fixture, and its own filter term runs the inherited tests.
- Each fallback in Decision 3 yields whole with its reason on stderr.
- A closure that selects no class from a non-empty seed runs no test command for any listed tier that opts in with `shared_build`, and runs the build step once, inside the first such tier's slots, failing when the build fails; each listed tier that does not opt in runs whole and prints the reason on stderr, whether or not another listed tier opts in.
- With a non-empty `inert_paths`, a diff touching only inert paths runs no tier but a `carry: false` one, which runs whole, under each of the three scopers at narrowed scopes, and the `go` scoper still plans its guard step; an untracked non-inert file keeps the scope whole; with the empty list every scoper behaves as today; an empty diff under `go` or `none` behaves as today, and under `dotnet` yields the empty scope and runs no build unless a listed `carry: false` tier opts in with `shared_build`, which runs the build before it.
- At a whole-scope gate, a tier whole-green on an ancestor whose diff to the clean tree is inert-only is carried and a whole entry is written for the current tree; with a non-inert path in that diff, a dirty tree, `carry: false`, or a changed `inert_paths` list, it runs.
- `{filter}` renders the tier filter alone for every invocation the `dotnet` scoper did not narrow to a selected project with its terms (whole scope, a whole-running tier inside a narrowed `dotnet` call, a command lacking `{targets}`, explicit targets, darn's tree-scope targets, a narrowed or `--card` `go` scope and the guard step), and nothing there without a tier filter; for an invocation the `dotnet` scoper narrowed to a selected project it renders `(<tier>)&(<terms>)`, or the terms alone without a tier filter; a filter past `greenCommandBudget` runs that project with the tier filter only.
- A narrowed `dotnet` pass writes no green entry, also when the hub's `all_targets` is its single test project, and a later whole gate on the same tree runs the tier.
- A changed `filter`, `shared_build`, `carry` or opted-in `build` declaration stops a tier from carrying.
- A `filter` on a tier without `{filter}`, `shared_build` with an empty `build`, and a non-empty `build` missing `command` or `timeout_min` are refused at load naming `lyx config gate`.
- The build step runs once per call before the first opted-in tier that runs, inside its slot hold, with the tier's timeout counted after it, and not at all when every opted-in tier is carried or has an inert-only or empty-diff empty scope; a failed build fails the remaining opted-in tiers without running them while a tier not opted in still runs, and the call exits with the first failed run's code in plan order, the build counting as a run before the opted-in tier it precedes.
- A build timeout prints its own timeout line, which `verifytree` reads as timed out; the build's run record is named and reported apart from tier runs.
- A failed build at Merge-In writes a landing failure record with `build: true` and no failing tests, and the blocked opted-in tiers print no failed `tier<N>` summary line.
- A `carry: false` tier runs whole under a computed or `--card` scope, is never carried and never writes a green entry.
- A whole pass of two `vstest` tiers on a tree carries both at a later gate on the same tree under the `dotnet` scoper.
- A `tier1` test drives one scoped `lyx gate test` run over a small fixture solution with `dotnet` and checks that only the selected class's tests ran; on Linux, with node reuse and the shared compiler at their defaults and the fixture tiers' and build's `timeout_min` below the SDK's node idle timeout, it checks that the call passes rather than timing out, which a node or compiler server holding the output pipe until its idle timeout would make it do.
  The fixture holds at least two projects MSBuild builds in parallel, and the fixture's build command passes `-m`, so an out-of-process node starts.
  The idle node outlives the test until the SDK's idle timeout, and the test names it as process-global state it leaves, since `dotnet build-server shutdown` would also end other gate runs' nodes.
  It restores with no network source, `--source` naming the local NuGet global-packages folder, and it skips, naming the reason, without `dotnet` or an SDK, or when that restore finds no test packages.
- Existing gate behaviour under `go` and `none` with none of the new keys is unchanged, held by the existing gate tests.

## Notes for the plan writer

- `gatecli.computeScope` returns early for any scoper but `go`; the `dotnet` branch and the inert check sit there, and `diffBaseGroups` gives the per-tier base the `none` scoper's inert check also needs.
- `greenRun.recordPass` decides a whole pass from targets alone today; the scope-filter rule in Constraints lands there, and `greenRecord.covered`, `greenRecord.newestWholeAncestor` and `tierFingerprint` hold the carry and fingerprint changes.
- `planparser.CardPackages` is Go-shaped and `diskPathForRef` maps only self glyphs to paths; the card seed function reads member glyphs' owner chains instead.
- The `go` scoper's "a `go-json` tier ran no test in a package" check stays `go-json`-only.
- The vet and guard steps stay `go`-scoper-only; `--guards` under `dotnet` is refused naming the scoper, as under `none`.
