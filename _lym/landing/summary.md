# Red-first gate judges C# tests under the dotnet scoper

`lym gate red-first` now runs a C# diff's changed tests on the merge base.
Before, it passed exit 5 on every hub whose scoper was not `go`, so a .NET run got no red-first protection
and a writer's handoff halted until the parent recorded a "No test seam" waiver by hand.
Under the `dotnet` scoper the gate selects the tests, runs them on the base, and judges red or green as it does for Go.

Closes #208

## What changes

- The gate reads the changed `.cs` files of the diff and parses the test methods in each, base against working tree.
  A method counts as changed when it is new or its source text differs.
  `internal/dotnetimpact` gains `ChangedTests`, backed by a per-method read in its source parser.
- It groups the tests by the nearest `.csproj` above each file and runs each project once, by fully qualified name
  (`FullyQualifiedName~` filter terms), through the key's tiers that take `{targets}`.
- The run happens in a checkout of the base written out from the git objects (`gitrepo.WriteTreeAtRevision`, reached through `fabricengine.ExportCodeTreeAt`).
  The changed files of the tests' own projects keep their working-tree text there; every other file keeps the base's text.
- A project whose tests do not build on the base is red, with the first compile error in the project's files as the reason.
  This covers a test that reads a member the change adds.
- `gateslot.RedFirstMode` replaces the `HasGoCoverageMaps` check in the verb:
  `go` keeps the build overlay, `dotnet` takes the checkout path, `none` stays unsupported.
- `gateslot` gains parsers of the tier output (`TierDiscovered`, `TierFailureNames`) that the verdict reads.
- The darn recipe comment, the writer stencil, `docs/overview.md` and the package docs describe the C# behaviour.

## Exit codes under the dotnet scoper

- 0: a changed test failed on the base, or its project did not build there.
- 3: every changed test passed on the base.
- 4: the diff changes no test method.
- 5, which a red-first gate passes: no changed test can be selected by name, as a method of an abstract class;
  a test file's project cannot be told; no tier of the key takes `{targets}`;
  or a project's run discovered no test, because its filter selected none or a restore or build failure ended it.
- 1: tests ran and none was judged, or a refusal.

Exit 5 is deliberate where the writer cannot act on the cause.
The gate halts only when the writer can make a test show the bug.

## Design decisions

- A checkout of the base, not an overlay: .NET has no build overlay, and a checkout keeps the working tree and its build output untouched.
- A test that reads a member only the change adds fails the compile on the base.
  Compile errors are matched only under the test project's directory, so a failure elsewhere in the solution is not read as red.
- Red is judged from the names the failure report lists, matched by method with the argument list of a parameterized test dropped.
  An exit code 0 with a summary that discovered no test is unjudged rather than green.
- Tier runs use `noCarry`, so the base's checkout does not inherit the carried state of a head run.

## To check by hand

- Run `lym gate red-first` once on a real .NET hub, such as NORCE Models, with a fix whose test fails on the base.
  The integration tests use a fixture project and the unit tests stub the tier run;
  neither has been run against that solution's own test command and tier configuration.
- The gate's own test suites were not run as part of writing this description.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
