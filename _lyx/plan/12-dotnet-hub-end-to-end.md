# Card 12 — a real dotnet run end to end, and the .NET hub guide

**Create:**
- `docs/dotnet-hub.md`
- `plan:internal/gatecli#TestGateTest_DotnetFixture` -> `func TestGateTest_DotnetFixture(t *testing.T)`
- `plan:internal/gatecli#writeDotnetFixture` -> `func writeDotnetFixture(t *testing.T, dotnet string) (repo, packages string)`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/gatecli#startGate`
- `internal/gatecli#gateRun`
- `internal/gatecli#runningGate`
- `internal/gatecli#runRecordOf`

**Intent:**
Prove the scoped run against a real `dotnet` and write the guide a .NET hub sets itself up from (the Scope's documentation bullet, Decisions 4 and 7 to 10).

`TestGateTest_DotnetFixture` is `tier1`, in a new test file of `internal/gatecli`.
It skips, naming the reason, when `dotnet` is not on `PATH`, when `dotnet --version` fails as on a host with no SDK, or when the offline restore probe finds no test packages.
The probe is a `dotnet restore --source <packages>` of a copy of the test project's package references in a `t.TempDir` of its own, outside the fixture worktree, so a missing package skips the test and the probe leaves no `obj/` in the tree the gate diffs.
`writeDotnetFixture` writes the fixture with `plankit.Repo`, outside every hub, then runs `git init` and commits it through `gitkit`, holding:
a solution with two production projects and one NUnit test project referencing both, so MSBuild builds at least two projects in parallel and an out-of-process node starts;
a `Circle` type in one project and a `Square` type in the other;
`CircleTests` and `SquareTests` test classes, each naming its type;
a `.gitignore` naming `bin/`, `obj/` and `.lyx/`, so the build's outputs and the gate's own run records outside every hub are no untracked change the `dotnet` scoper reads as non-inert;
and the worktree's own `_lyx/config/gate.yaml` with `scoper: dotnet`, `build: {command: "dotnet build Fixture.sln -m --source <packages>", timeout_min: 5}` and a `tier0` of `counter: vstest`, `shared_build: true`, `timeout_min: 5` and the command `dotnet test {targets} --no-build {filter}`, both timeouts below the SDK's node idle timeout.
`<packages>` is the NuGet global-packages folder `dotnet nuget locals global-packages --list` names, so the restore uses no network source, and the package references pin versions the test finds there.
The test changes `Circle` on top of the fixture's commit and runs `lyx gate test --base <commit>` through `startGate`.
It asserts the call exits 0 rather than timing out, which a node or compiler server holding the output pipe until its idle timeout would make it do, that the run record's counts are `CircleTests`' alone, and that the tier's command named the test project and `--filter` holding `CircleTests` and not `SquareTests`.
Its doc comment names the MSBuild node it leaves as process-global state that outlives the test until the SDK's idle timeout, since `dotnet build-server shutdown` would also end other gate runs' nodes.

`docs/dotnet-hub.md`, one sentence per line, sets up a .NET hub with the Models repository as its worked example:
`scoper: dotnet` and what it narrows, with the Bound of Decision 2;
tiers with `{targets}`, `{filter}`, a `filter` such as `Category!=Slow` that a landing tier uses only to exclude categories, and `shared_build: true` with `--no-build`;
the `build` step for the Models solution with `-p:RunSlowTests=true -p:RunBaseline=true`, and the restore it does implicitly;
`inert_paths` and its Bound;
the recommended gate keys of Decision 10;
the timing tier of Decision 8, `slots: all`, `manual: true` and `carry: false` run only through a direct `lyx gate test --wait --tier N` call, kept out of darn's gate keys and every tier's `covers`;
`refuse_raw: ["dotnet test"]`;
`--ignore-exit-code 8` or `TESTINGPLATFORM_EXITCODE_IGNORE: "8"` for every `{filter}` tier on Microsoft.Testing.Platform;
the warm-process behaviour on Linux and Windows and `MSBUILDENSURESTDOUTFORTASKPROCESSES` and `MSBUILDNODEWINDOW` left unset (Decision 7);
the repo-rules lens of Decision 9 as a `lenses.yaml` entry added to the `review-code` fan;
and taking a `lyx gate report` baseline on OSL-1046 before installing the binary this task lands.

`docs/overview.md`'s gate summary gains the `dotnet` scoper, the `{filter}` placeholder, the shared build step, `inert_paths` and `carry: false`, and links `docs/dotnet-hub.md`.

**ImpactSummary:** a new test and a new doc; `docs/overview.md` gains one paragraph and a link, which `docslink_test` checks.
