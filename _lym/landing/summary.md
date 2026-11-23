# Red-first gate runs a C# hub's changed tests on the merge base

`lym gate red-first` now checks bug fixes in a hub whose scoper is `dotnet`, such as a .NET solution tested with NUnit.
Before, such a hub exited `gateslot.RedFirstUnjudgedExit` and the gate passed, so a C# run had no red-first check at all.
Now the verb takes the test methods the diff adds or changes, runs them on the merge base's production code through the hub's own tiers, and judges them red or green as it does for Go.

Closes #208

## How it works

- **Selection.** `dotnetimpact.ChangedTests` compares the base and head text of each changed `.cs` file.
  It returns every test-attributed method (`[Test]`, `[TestCase]`, `[Fact]`, `[TestMethod]` and the rest the scoper already knows) whose text, from its first attribute list to the end of its body, matches no base method of the same class and name.
  Each comes with its class's fully qualified name and the `FullyQualifiedName~<Class>.<Method>` term that selects it.
  The source reader in `internal/dotnetimpact/source.go` now records each test method's name and text on its type.
- **Base checkout.** Fabric's new `ExportCodeTreeAt` writes the base commit's tree into a `lymtemp` directory, through gitrepo's `WriteTreeAtRevision`.
  Every changed file whose nearest `.csproj` is the project of a selected test keeps its working-tree text there, including the `.csproj` itself and test data.
  Every other file, production code included, keeps the base's text.
- **Run.** The verb runs the gate key's tiers that hold `{targets}`, once per test project.
  The one target is the project's absolute path in the checkout, and the tests' terms fill `{filter}`.
  A new `checkoutRoot` field on the gate test request replaces the worktree root, so the shared build and the tiers run in the checkout.
  A new `terms` field carries the filter terms onto explicit targets.
  The run is `noCarry`, so it writes no green record.
- **Verdict.** A test is red when the failure report names its method under a tier run: bare, with arguments, or fully qualified.
  The new `gateslot.TierFailureNames` reads those names.
  With no test named, a C# compile error (`<file>.cs(l,c): error CSnnnn`) in a file of the test project makes the project's tests red.
  A test that reads a member only the change adds does not build on the base, so it counts as red.
  The envelope then carries `build_errors`.
  Exit codes keep their meaning: 0 red, `RedFirstGreenExit` all green, `RedFirstNoTestExit` no changed test, and 1 for a run that judged none.

## Design decisions

- **What still exits unjudged.** The verb still exits `RedFirstUnjudgedExit`, which the gate passes, in four cases:
  every changed test is a method of an abstract class, which no filter selects by its own name;
  a test file has no `.csproj` above it;
  the nearest directory holds two `.csproj` files;
  or no tier of the key holds `{targets}`.
  None of these is something the writer can mend in the bug fix, so holding the handoff would only halt it.
  This follows the issue's rule that the gate halts only on what the writer can act on.
- **What exits 1.** A build failure outside the test project, such as base production code that does not compile or a failed restore, judged none of the tests.
  It exits 1 as a Go run that judged none does, which the gate treats as terminal.
- **A checkout, not an overlay.** C# has no build overlay.
  Exporting the base tree keeps the writer's worktree untouched, and building there reads the repository's own `Directory.Build.props`, solution and project references.
  The shared build step builds the checkout, so a tier that runs `dotnet test --no-build` works unchanged.
- **Method-level terms.** The terms use `~` (contains) as the issue asks, so `Ns.C.Method` also matches `Ns.C.MethodTwo` and a parameterized `Method(1)`.
  The over-selection only runs tests of the same class on the base.
  Only a selected test's failure counts as red, so an extra test that passes changes nothing.
- **Changed-test granularity.** The comparison reads raw method text, so an edit inside a string literal or a comment marks the test changed.
  Such a test then passes on the base and gives the green verdict, which is correct.
- **`filterTerm` and `ChangedTest` share one spelling of a class's qualified name** (`qualifiedName` in `internal/dotnetimpact/changedtests.go`).

## Bounds

- The method reader takes the identifier before a test member's first `(` as its name.
  A tuple return type such as `public (int, int) M()` therefore reads the wrong name.
  A `[Test]`-attributed field whose initializer calls a method is read as a method.
- A failed test name matches by method name.
  A filter-less tier, whose command holds no `{filter}`, runs the whole project, and a same-named failing method of another class would then read as red.

## To check by hand

- `TestGateRedFirst_Dotnet` in `internal/gatecli/redfirstdotnet_integration_test.go` drives the verb over a real .NET solution in a hub.
  It covers three cases: a red test, a test that reads a member only the change adds, and a green test.
  It skips on a machine without `dotnet` or without the NUnit packages in the global-packages folder.
  Confirm it ran rather than skipped on a .NET machine.
- The compile-error match reads MSBuild's classic console format.
  A tier whose build prints through the terminal logger, which is used only on an interactive terminal, would not match.
- No run was made against NORCE Models itself.

## Merge danger

**Door:** two-way
**Blast radius:** gate
