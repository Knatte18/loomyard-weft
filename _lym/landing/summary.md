# Gate: name skipped dotnet tests, fix testmap's way forward, deflake two

Three gate bugs, one commit each.

Closes #126
Closes #127
Closes #118

## What changes

### A gate verb run from a records worktree acts on its code worktree (#126)

Run from a records worktree, `lym gate testmap` resolved that worktree itself.
This happens whenever the cwd lies under a worktree's `.scratch`, which links into the records worktree.
Its git main worktree is the prime's records worktree, so the refusal's way forward named `loomyard-weft`, where the verb refused again for want of a `go.mod`.

- `fabricengine.CodeLocation` (new, in `weftwiring.go`) maps a records worktree's `Location` to the code worktree it pairs with, through `WeftWarpSlug` and `lymcwd.NamedWorktree`.
  Any other `Location` comes back unchanged.
- `gatecli.resolveHub` applies it, so every gate verb resolves a records worktree to its code worktree.
  The testmap refusal then names the prime's code worktree.
  Run from the prime's records worktree, the verb acts in the prime.
- `hubgeom.PrimeWorktreePath` has one caller, this refusal.
  Given a code worktree it returns the code prime, so it needs no change.
  `lym gate redundant` had the same mix-up through `resolveHub`: it looked for the module in the records worktree.
  It is fixed by the same change.
- Two rows in the testmap refusal test pin this: one from a pair's records worktree and one from the prime's.

### A dotnet run that reported no test is named (#127)

A test project that sets `IsTestProject` false unless an MSBuild property is passed builds and exits 0 under `dotnet test` with no test run.
The gate passed it with no word.

- `runInvocations` (`tierrun.go`) compares the `vstest` counter's discovered count before and after each passing invocation.
  When the invocation reported no test, `noTestNotice` returns a line for stderr and the run log.
  The line names the tier and the project, or the run directory when there are no targets, plus the class filter terms the scope gave it.
  Its way forward is running the tests by hand with `dotnet test <project> -p:<property>=true`, using the property the project file's `IsTestProject` condition names, and checking the summary line.
- The tier still passes.
  A project that opts out on purpose, such as a slow suite gated on a setting, must not fail every run.
  The line is the visible signal the issue asks for, and it is also the check that flags a zero-test run.
- The darn writer's make-change list and the webster implementer prompt now say what such a line means.
  They also allow running those tests by hand as the line names, as the one exception to "never a raw test command".
- `TestRunTierNoTestNotice` pins the line for a narrowed project and a whole run, and pins its absence for a project that reported a test and for a `go-json` tier.

### Two flaky gatecli subtests made deterministic (#118)

- `TestGateTest_DetachedRun`, "a passed lease ends a --wait gate line through its child…": the lease passed one second after the one-second cap.
  The line's child, a nested `lym gate test --wait`, had to start its tier within those two seconds; passing runs took 2.1 to 3.9 s.
  On a loaded machine the runner could end the line before the tier ran, and the wait for the tier invocation then timed out.
  The subtest now gives a minute's grace, waits for the tier invocation, and expires the lease itself with `expireLease`.
- `TestRedundancyStep`, "ABrokenLymBuildAndACorruptMap…": the subtest asserted `TestBetaNewOne` contained under a 3-second per-test timeout.
  The timeout had to stay that short because `TestDropsHangs` runs to it.
  The subtest now marks `TestDropsHangs` kept for its run, restoring the file in cleanup, and raises the timeout to 20 seconds, so load cannot time out a judged test.
  The subtest's run time drops by the 3 seconds `TestDropsHangs` used to take.

## Design decisions

- The records-to-code mapping lives in `fabricengine`, the owner of the records-sibling naming, and is applied in `resolveHub` rather than only in the testmap refusal.
  A gate verb run from a records worktree finds no module there, so every gate verb needs the mapping.
- The no-test line is advisory, not a failure, because opting out of a test project is legitimate.
- No archived log holds the failure output of either flaky subtest.
  The one archived `ABrokenLymBuild…` failure, from 2026-11-09, is a cascade from an older version of `TheReportMatchesACommitOfItsTree…`, since fixed.
  The fixes remove the timing dependence the code shows; neither was reproduced under load.

## To check by hand

- On a hub with a .NET project that disables itself as a test project, a `lym gate test` run over it prints the new line naming the project.
- `TestRedundancyStep`'s broken-build subtest: if it still flakes, look at its report errors first; a judged test's timeout is no longer a possible cause.
