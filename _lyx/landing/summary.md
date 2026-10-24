# Scoped build and test for .NET hubs through a dotnet gate scoper

A .NET hub's verify gates now build once and test only the test classes a change touches, and keep the whole solution to the landing gates.
The task also delivers the remaining wants of the Models verification protocol: fewer full-suite runs, an exclusive timing tier, one build shared by several tiers, and repo rules for code review.

Closes #609

## What changes

`gate.yaml` gains the following:

- `scoper: dotnet`, beside `go` and `none`.
- A `{filter}` placeholder and a tier `filter` key.
- A hub-level `build` step.
- A tier `shared_build` key.
- A hub-level `inert_paths` list.
- A tier `carry: false` key.

`docs/dotnet-hub.md` is new and sets up a .NET hub with the Models repository as its worked example.
`docs/overview.md` and the package docs of every touched package are updated.

## Design decisions

- **The map.**
  The new `internal/dotnetimpact`, the sibling of `internal/impactset`, lexes every `.cs` file under the tier's `dir` and closes the changed types over the type names other files spell.
  A test class is selected when its file changed, is a card Target, or names a type in the closure.
  Each selected test project runs once, with one `FullyQualifiedName~` term per selected non-abstract class; nested classes join with `+` and a derived fixture gets its own term for inherited tests.
  Card seeds come from a new exported function of `internal/planparser`.
- **What it misses.**
  A name match over-selects safely but misses references that never spell the type: dispatch through a base class or interface, extension methods, `var`-typed factory results, reflection and source-generated code.
  The gates that run whole catch these.
- **Fallbacks.**
  The scoper runs whole, with the reason on stderr, for a changed path that is neither `.cs` nor inert (`.csproj`, `.props`, `.sln`, test data), a `.cs` file with no `.csproj` ancestor or no declared type, an unusable base, or a failed git or file read.
  A closure that selects no class from a non-empty seed runs no test command for tiers that opt in with `shared_build`, only the build, and runs other tiers whole.
- **`{filter}`.**
  It renders `--filter <expr>`, quoted for the platform shell.
  The expression is the tier's `filter` alone for any invocation the scoper did not narrow, and `(<tier filter>)&(<terms>)` for one it did.
  A filter that renders past `greenCommandBudget` falls back to that project's whole invocation.
  `gateslot.LoadConfig` refuses a `filter` on a tier whose command lacks `{filter}`.
- **Green record.**
  A narrowed `dotnet` pass writes no green entry, so a filtered pass never stands in for a whole run.
  `carry`, `filter`, `shared_build`, a non-empty `inert_paths` and the `build` declaration join the tier fingerprint only when set to a non-default value, so existing tiers' fingerprints and green entries are unchanged.
- **`inert_paths`.**
  A diff touching only inert paths runs no tier but a `carry: false` one, under every scoper.
  At a whole gate a tier whole-green on an ancestor with an inert-only diff to the clean tree is carried.
  Loomyard keeps the empty list because its tests read Markdown.
- **Shared build.**
  `lyx gate test` runs the `build` command once, inside the slot hold of the first opted-in tier that runs, before that tier's command.
  The build has its own `timeout_min`, run record, log and summary line.
  A failed or timed-out build fails the opted-in tiers it blocks without running them, while other tiers still run.
  A Merge-In landing failure record gets `build: true` and no failing tests, so the landing fixer is pointed at a compile break.
  The build restores implicitly, so no separate restore step exists.
- **Warm processes.**
  lyx sets nothing that disables MSBuild node reuse or the shared compiler server.
  On Windows the kill-on-close job object ends nodes with each run, so the saving there is the single build and the `--no-build` runs.
- **`carry: false`.**
  A tier with it runs whole wherever a `lyx gate test` call lists it and never carries or writes a green entry.
  With `slots: all` and `manual: true` it is the exclusive timing tier.
  A verify site's verified-tree record, explicit targets and `covers` can still skip or narrow it, and `docs/dotnet-hub.md` tells a hub how to keep it out of those.
- **Repo rules.**
  The Models rules are lens prose for the hub's own `lenses.yaml`, documented in `docs/dotnet-hub.md`; lyx gains no rule engine.
- **Gate keys.**
  `docs/dotnet-hub.md` recommends scope `card` or `diff` for the early gate keys and scope `all` for `merge-in` and `finalize`, so Merge-In is the one planned whole-solution run.
  The template's keys and loomyard's own are unchanged.
- **PATTERN.**
  No entry changes; the `PATTERN-gate-slot` line on `-trimpath` is narrowed to Go builds.

## To check by hand

- Verbatim-string lexing in `internal/dotnetimpact/lex.go`: a doubled quote inside a verbatim string was lexed as the start of a raw string.
  Follow-up commits correct it and the `record` declaration follower; check a source with `@"a ""b"" c"` and `$@"..."` literals for identifiers or holes read wrongly.
- Not run: the first real C# run on OSL-1046.
  Models is never cloned elsewhere, so take a `lyx gate report` baseline there before installing the binary this task lands, then measure build and test minutes.
- Not run: a hub on Microsoft.Testing.Platform.
  Its tiers whose commands hold `{filter}` need `--ignore-exit-code 8` or `TESTINGPLATFORM_EXITCODE_IGNORE: "8"` so a run that matches no test exits 0.
- Open risk: the closure may select classes in every test project for a widely named type, and one `dotnet test` per project in sequence can then cost more than one solution-level run.
  The whole-solution build may likewise cost more than building one test project on a large solution.
- Out of scope: the numerical baseline (note `regression-baseline-verify`) and the hubs' own `gate.yaml` and `lenses.yaml`.
- The verify run's first attempt failed on `cmd/lyx.TestCLIForm_EveryCommand`, in a package no card touched; a fix commit brought the `gate test` help text back under its 2000-byte cap.
  Check that test passes.
