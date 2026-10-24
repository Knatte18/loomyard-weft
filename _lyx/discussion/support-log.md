# Support log — dotnet-scoped-verify

## Interview

Autonomous run: no operator answered, and no advisor was seated.
Every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Read the board entry, every related entry (`regression-baseline-verify`, `csharp-sandbox-hub`, `bench-osl1046-xdr`, `run-profiles-ladder`, `umbrella-anchor`), the gate packages' docs, `computeScope`, `CardPackages`, the lenses, runproc and proc docs, the `cs` sandbox hub's config, quarry v0.4.0's public API, and the PATTERN backgrounds for gate-slot, test-speed and leaf-packages.
Findings: the scoper seam exists (`go`, `none`); the `vstest` counter exists; the green record already carries every tier per tree; the pool's `slots: all` drain is a hub-wide exclusive hold; `lenses.yaml` is hub-tunable; quarry answers declarations, not references.

### Turn 2 — facts researched

- VSTest filter grammar combines conditions with `&`, `|` and parentheses (Microsoft Learn, "Run selected unit tests").
- MSBuild node reuse is on by default for multi-node builds; the MSBuild server is opt-in through `DOTNET_CLI_USE_MSBUILD_SERVER`; nodes reset their working directory after a build, while server-loaded task assemblies stay locked (MSBuild team blog "Node reuse in multiproc MSBuild"; Microsoft Learn "MSBuild server"; dotnet/msbuild issue on server-held assemblies).
- NuGet no-op restore skips work when the `project.nuget.cache` hash of its inputs is unchanged.

### Turn 3 — design batch (auto-picked)

1. Test map: name closure over C# sources selecting test classes (recommended) / project graph only / coverage-based impact. Picked the closure.
2. Missed tests: whole gates catch them (recommended) / cards name tests under `Uses:`. Picked whole gates.
3. Filter composition: lyx composes tier filter and scope terms through `{filter}` (recommended) / hub writes the filter in `{args}`. Picked `{filter}`.
4. Build sharing: hub `build` step plus tier `shared_build` (recommended) / per-tier build in each command. Picked the build step.
5. Restore: implicit in the build command (recommended) / lyx restore fingerprint. Picked implicit.
6. Doc-only: hub `inert_paths` list across scopers plus a green carry (recommended) / Markdown hard-coded in the `dotnet` scoper. Picked `inert_paths`.
7. Exclusivity: `slots: all` manual tier plus `carry: false` (recommended) / a new lock verb. Picked the tier.
8. Repo rules: documented hub lens (recommended) / a lyx rule checker. Picked the lens.

### Turn 4 — lens review

Every fork returned a report.
Checked `internal/proc/tied_windows.go` and `namedjob_windows.go` myself before judging: the Windows tied child runs in a kill-on-close job, confirming bound-breaker 1.

- bound-breaker 1 (BLOCKING, D7 Windows nodes die with the job): fixed; D7 now states per-platform behaviour and why no breakaway is added; the Windows teardown-lock risk is removed, since nodes cannot outlive the job.
- bound-breaker 2, existing-paths 1, walkthrough 2, interactions F2 (BLOCKING, filtered pass recorded whole): fixed; merged into one Constraint: a run with scope filter terms or fewer projects writes no green entry; new acceptance criterion.
- bound-breaker 3, walkthrough 1 (BLOCKING, card member glyphs name no file): fixed; card seed takes owner type names from member glyphs and `plan:` handles, and a Target yielding neither file nor type runs whole.
- bound-breaker 4 (no-type `.cs` file seeds nothing): fixed; added to D3's fallbacks.
- bound-breaker 5 (empty scope skips the build, hiding a compile break): fixed; an empty test scope from a non-empty seed still runs the build step.
- bound-breaker 6, walkthrough 4, interactions F6 (build eats the tier timeout): fixed; the tier's clock starts after the build.
- bound-breaker 7 (inert_paths outside the fingerprint): fixed; a non-empty list joins every tier fingerprint.
- bound-breaker 8 (attribute spellings): fixed; D2 and an acceptance criterion name the suffix, qualified and list forms.
- bound-breaker 9, interactions F8 (guard step on inert-only empty scope): fixed; the `go` scoper keeps planning it.
- existing-paths 2 (fingerprint misses `filter`, `shared_build`, `carry`): fixed in D8 and acceptance criteria.
- existing-paths 3 (template must hold `build`): fixed; template seeds `build: {}`, empty means no build.
- existing-paths 4, interactions F7 (run record naming and report counting): fixed in D6.
- interactions F1 (empty diff counts as all inert; none scoper changed with empty list): fixed; inert rules need a non-empty list and at least one changed path.
- interactions F1 second half, walkthrough 3 (timing tier narrowed or emptied): fixed; a `carry: false` tier runs whole under every form but explicit targets.
- interactions F3, walkthrough 5 (build `{args}`): fixed; the build command takes no placeholder.
- interactions F4 (build timeout reading): fixed; own timeout line that `verifytree` reads as timed out.
- interactions F5 (exit code after build failure): fixed; the call exits with the first failed run's code in plan order, the build counting as a run before the opted-in tier it precedes.
- walkthrough 6 (build output): fixed; passes through like a tier's.
- walkthrough 7 ("would pass" wording): fixed.
- goal-scope 1 (three whole gates against "once at the final gate"): fixed; new D10 recommends `webster: diff` for a .NET hub so Merge-In is the one whole run and Finalize carries.
- goal-scope 2 (restore once per run reinterpreted): fixed; D6 states how the want is met by no-op restore.
- goal-scope 3 (before-measurement baseline): fixed; the docs page tells the operator to take a `lyx gate report` baseline before installing.
- goal-scope 4 (pin that nothing disables node reuse): rejected; nothing sets those variables today, and a test for an absent string pins no behaviour.
- goal-scope 5 (inert path base): fixed; patterns are relative to the worktree root.
- goal-scope 6 (Models named in public docs): kept, recorded as an auto-mode assumption, since issue #609 names it.
- board-fit 1, 2 (note `regression-baseline-verify` needs `carry: false` and no narrowing): fixed; D8 names the note and `carry: false` forces whole.
- board-fit 3 (F18 capture and the build step's ordering): rejected; F18's own join-before-Webster ordering is that note's design, and nothing here changes it.
- board-fit 4 (name note `csharp-sandbox-hub`): fixed in Scope.
- board-fit 5 (test-tiers' deliverables stated without evidence): fixed; Scope now cites the board entry's assignment only.
- board-fit 6 (toolchain in `gate.yaml` follows the 2026-10-21 decision): fixed in Q11.
- leanness 1, 2 (rejected alternatives in D2 and D6): fixed; cut to rationale.
- leanness 3 (D9 restates): fixed; its one new fact moved to Constraints and D9 removed.
- leanness 4 (D11 restates Scope): fixed; folded into Scope.
- leanness 5 (D7 explanation): rejected in part; D7 now carries the Windows decision, which the plan writer needs.
- leanness 6 (auto-mode assumptions restate decisions): fixed; only real assumptions kept.
- leanness 7 (Scope restore bullet): fixed; cut.
- leanness 8 (cs hub note in plan-writer notes): fixed; cut.
- leanness 9 (duplicate constraint and acceptance criterion): fixed; the acceptance criterion kept.
- leanness 10 (repo-rule texts, now D9): no change, as the lens concluded.

## Rejected alternatives

- Project-graph-only map: picking test projects from `ProjectReference` alone narrows little when one test project serves many production projects (operator, 2026-10-12).
- Coverage-based test impact analysis: needs a coverage run per tree and a stored map that goes stale; costs more than the runs it saves on a small suite.
- Naming convention alone (`FooTests` for `Foo`): misses every test of a caller and every test named otherwise; the closure covers the convention's hits anyway.
- A quarry references query: quarry v0.4.0 exposes glyphs, resolve, toc and expand only, so it would need new quarry work for no gain over a lexer.
- Tests named under a card's `Uses:`: adds plan-writer burden for misses the whole gates already catch.
- Filter in `{args}`: `dotnet test` takes one `--filter`, so a tier filter and a scope filter cannot both be passed.
- Building per selected test project: `dotnet build` takes one project per call, so several projects re-evaluate shared references; one incremental solution build skips untouched projects.
- A lyx-kept restore fingerprint: adds a record and a stale-`obj/` failure for seconds of no-op restore.
- Hard-coding `*.md` as doc-only in the `dotnet` scoper: lyx hard-codes no language, and loomyard's own tests read Markdown.
- A new exclusive-lock verb: duplicates the pool's drain.
- A lyx rule checker for repo rules: the board entry says these are not tool features.
- Windows job breakaway for MSBuild nodes: a job that permits breakaway lets test processes escape the tie too, and MSBuild does not request breakaway itself.
- Keeping the Webster end gate whole for a .NET hub: Merge-In already runs whole over the merged parent, so a whole Webster gate would be a second full run.

## Review rounds

### Round 1 — fix

Facts researched for F1, F8 and F9:

- MSBuild's node launcher redirects a node's stdio on Unix and gives it invalid standard handles with `bInheritHandles: false` on Windows (dotnet/msbuild `NodeLauncher.cs` at `vs17.12`); in main's launcher, newer than `vs17.12`, the escape hatches `MSBUILDENSURESTDOUTFORTASKPROCESSES` and `MSBUILDNODEWINDOW` leave a Unix node's stdio unredirected.
- Roslyn's `TryCreateServer` starts VBCSCompiler the same way (dotnet/roslyn `BuildServerConnection.cs` at `release/dev17.12`).
- The test platform's `VSTestTask` and its build call run only when `IsTestProject` is `true` (microsoft/vstest `Microsoft.TestPlatform.targets` at `v17.12.0`), so a gated Models project evaluated without its property tests nothing and succeeds.
- VSTest's `TreatNoTestsAsError` defaults to false (Microsoft Learn, ".runsettings"); Microsoft.Testing.Platform exits 8 on zero tests run, ignored by `--ignore-exit-code 8` or `TESTINGPLATFORM_EXITCODE_IGNORE` (Microsoft Learn, "Microsoft.Testing.Platform exit codes").
- Models is .NET 9 with NUnit, and its gated projects set `IsTestProject=false` unless their property is passed (board entry).

## Question ledger

- Q1 Test map — auto-pick: type-name closure selecting test classes. Resolved.
- Q2 How a missed test is caught — auto-pick: the whole gates `merge-in` and `finalize`, with `webster` at scope `diff` per Q12. Resolved.
- Q3 Filter composition — auto-pick: `{filter}` placeholder and tier `filter` key. Resolved.
- Q4 One build for several tiers — auto-pick: hub `build` step, tier `shared_build`. Resolved.
- Q5 Restore once per run — auto-pick: implicit no-op restore in the build command; no lyx restore step. Resolved.
- Q6 Doc-only diffs — auto-pick: hub `inert_paths` and an inert-only green carry. Resolved.
- Q7 Exclusive timing lock — auto-pick: `slots: all` manual tier with `carry: false`, called with `--wait`. Resolved.
- Q8 Warm build processes — auto-pick: keep SDK defaults, document the MSBuild server opt-in. Resolved.
- Q9 Repo rules — auto-pick: lens prose in the hub's `lenses.yaml`, shown in `docs/dotnet-hub.md`. Resolved.
- Q10 Measurement on OSL-1046 — auto-pick: operator measures after landing with `lyx gate report`; the run cannot reach Models. Resolved.
- Q11 Where the toolchain is declared — auto-pick: `gate.yaml` (scoper, tiers, build); `language.yaml` keeps skills and rename tools, following the operator decision of 2026-10-21 recorded on note `regression-baseline-verify`. Resolved.
- Q12 Gate-key scopes for a .NET hub — auto-pick: `webster` at `diff`, Merge-In the one whole run, Finalize carrying. Resolved.
- Q13 Warm nodes on Windows — auto-pick: no breakaway; the saving there is one build per call. Resolved.
