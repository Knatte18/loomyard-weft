# gate test: refuse or warn when a tier command drops requested targets

`lym gate test` no longer reports a silent pass for tests it did not run.
A tier command with neither `{targets}` nor `{test_targets}` runs whole and ignores the targets the call asked for.
Two cases hit this with no warning:

- An explicit target, such as `lym gate test <project> -- --filter ...`, ran the fast suite instead.
- A card or batch gate whose tests live in a second test project never reached that project.

Closes #105

## What changes

- **Explicit targets no tier takes are refused.**
  When every kept tier's command lacks both placeholders, `planTierRuns` refuses the call before any slot is taken.
  The refusal offers three ways forward:
  drop the targets, name a `--tier` whose command takes them, or have the prime run `lym config gate` to add `{targets}`.
  It closes with the usual clause for a session that is not the prime.
  A call where at least one kept tier takes the targets still runs; the others run whole.
- **The run-whole notice moves.**
  `gate test: the command of tierN holds neither {targets} nor {test_targets}, so it runs whole` now prints from `planTierRuns` instead of `runListed`.
  It also prints for a narrower scope, not only for explicit targets.
  It prints only once no refusal can follow, so a refused call prints none.
  The `ignoresTargets` field of `tierRun` is gone; the notice is the only thing it fed.
- **Unreached test projects are named under the dotnet scoper.**
  `unrunProjectLines` in `dotnetscope.go` checks the projects a narrowed share selected for a whole-running command.
  After the call it prints one stderr line for each project that no run of the call reaches.
  The line says the pass is not green for that project.
  Its way forward is the lowest declared non-manual tier whose command reaches the project, as `lym gate test --tier N [project]`.
  When no tier does, the way forward is the prime's `lym config gate` step.
- **The warning is output only.**
  It changes no exit code and no record.

## Design decisions

- **What counts as reaching a project** (`namesProject`):
  a run reaches a project when its names hold that project's file, that project's built assembly, or a solution file.
  It also reaches every project when its names hold no project, solution or built assembly file, as with a bare `dotnet test`.
  The names are the run's invocation targets, or for a command without a target placeholder the words of the command.
  Project paths are compared case-insensitively and with slashes normalised.
  A bare file name matches on file name, and a path matches when one is the other or ends at a directory boundary with it.
- **A command naming only other built assemblies reaches no project.**
  A built assembly is a specific target, so it is matched to the project of the same name instead of counting as "no files named".
- **The warning is not an error.**
  Unlike explicit targets, a card scope runs real tests in the other projects, so a refusal would block a legitimately partial gate.
  The line makes the gap visible without changing the verdict.
- The Go scoper is untouched; the unreached-project line applies only when `cfg.Scoper == "dotnet"`.
- `doc.go` documents the refusal, the notice and the unreached-project line.

## Tests

- `TestUnrunProjectLines` in the new `dotnetscope_test.go` covers the reach rules and the way forward.
- `TestPlanTierRuns` covers the refusal, the notice beside a tier that takes targets, and the notice under a narrower scope.
- `TestGateTest_Scopes` covers a `--card` call under dotnet that prints the line and still exits 0.
- The integration test for explicit targets on a command without `{targets}` is removed, as that call is now refused.
  The missing-build-dir test now reads its JSON envelope from the last stdout line, since the notice precedes it.

## To check by hand

None of these was exercised against a real multi-project dotnet hub.

- `namesProject` in `dotnetscope.go`: a built assembly (`.dll` or `.exe`) now reaches only the project it is named after.
  Check that this holds for assemblies whose name differs from the project file name.
- The `lym gate test` wiring in `runTest` in `run.go`: the dotnet-only call to `unrunProjectLines` after the run is covered by one integration case.
  Check the line appears in a real card gate and that a non-dotnet hub prints nothing.
- The targets refusal in `planTierRuns` in `tiers.go`: it ends with the role clause like the other refusals.
  Check the wording of the full refusal on a hub whose tier0 command lacks `{targets}`.
