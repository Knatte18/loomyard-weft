# Gate bugs: silent skips, a testmap refusal naming the weft, two flaky gatecli tests

`gate-bug-bundle` · Task · bug, gate, dotnet, stencil, cwd · running · Drift-Expect

Three gate bugs, bundled because they all touch the gate packages (operator, 2026-11-15).

## 1. The gate skips tests silently (issue #127)

A task agent had to run a project's slow test suite by hand, because the gate's `COMMONLIB_REPO` setting makes the gate skip those tests.
`dotnet test` on the slow-test project printed only restore lines and exited 0: no test ran and no message said why.
The project disables itself as a test project unless an opt-in MSBuild property is passed, so the run looked like a pass.
Neither the agent's prompt nor the harness README mentions it.

- Have the gate report which test classes it skipped and why, as a visible line in its output.
- Have the agent prompt tell agents how to run gate-skipped tests by hand.
- Consider a lym check that flags a `dotnet test` run that executed zero tests.

## 2. The `lym gate testmap` refusal names the weft (issue #126)

Run from a task worktree, `lym gate testmap internal/adoptcli` refused because the worktree is not the prime.
The refusal's way forward named the prime's weft directory (`loomyard-weft`), where the verb refused again: that directory holds no `go.mod`.
The way forward is built from `hubgeom.PrimeWorktreePath`, which returns the weft path where a code path is needed.

- Make the refusal name the prime's code worktree.
- Check the other refusals that take their way forward from `hubgeom.PrimeWorktreePath` for the same weft-for-code mix-up.

## 3. Flaky gatecli tests (issue #118)

Two gatecli tests failed a first verify run and passed or vanished on rerun:

- `TestGateTest_DetachedRun`, subtest 'a passed lease ends a --wait gate line through its child, which kills its tiers and frees its slots'
- `TestRedundancyStep`, subtest 'ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged'

Both exercise process, lease and slot handling and a lym build, which points to timing or shared-state dependence.
Find the race or shared state and make them deterministic.
