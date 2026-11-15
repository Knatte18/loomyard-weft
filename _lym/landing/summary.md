# Gate: report skipped tests, name the code worktree, fix flaky tests

Three gate bugs that all touch `internal/gatecli`.

Closes #127
Closes #126
Closes #118

## Silent test skips (#127)

A `vstest` tier whose project sets `IsTestProject` false unless an MSBuild property is passed builds, exits 0 and runs no test.
The gate passed it without a word.
Now a passing `vstest` invocation whose output reported no test prints a line to stderr and the run log.
The line names the tier, the invocation's targets (or its directory when it has none) and the class filter terms.
Its way forward is a hand run of `dotnet test <project> -p:<property>=true`.
That way forward is offered only while the hub's `refuse_raw` does not list `dotnet test`; otherwise the agent reports the skipped tests to its parent.

The agent stencils (`darn-fragment-make-change.md`, `webster-body-implementer.md`) now tell agents what this line means and how to run the skipped tests by hand.
The `gatecli` package doc describes the line and its limits.

Design decisions:

- The line reads only each invocation's total, so it names no MSBuild property and cannot see a disabled project beside a project that ran, or a gated class inside a project that ran other tests.
  The line states this limit itself.
- `runInvocations` takes a `notice` callback so the line reaches both stderr and the log.
- The optional lym check for a `dotnet test` run that executed zero tests is not added; the gate line covers the gate's own runs.

## `lym gate testmap` refusal names the code worktree (#126)

Run from a task worktree reached through its `.scratch` link, the cwd resolves to the weft sibling.
The refusal then named the prime's weft directory, which holds no `go.mod`.
`fabricengine.CodeLocation` maps a weft sibling to the code worktree it pairs with, and `resolveHub` applies it, since every gate verb acts on code.
The mapping fixes the refusal and any other gate verb whose way forward is built from `hubgeom.PrimeWorktreePath`, because they all start from `resolveHub`.
A location that is no weft sibling, or whose code worktree is absent, comes back unchanged.

## Flaky gatecli tests (#118)

Both tests were timing-dependent; no production code changes.

- `TestGateTest_DetachedRun`, the `--wait` lease subtest: a 1s grace could expire before the line's child started the tier, so the lease ended the line before its child ran.
  The grace is now 1m and the test expires the lease itself once the tier is held.
- `TestRedundancyStep`, the broken-lym-build subtest: a loaded machine could time out `TestBetaNewOne`.
  The subtest now uses a 1s second unit, a 20s per-test timeout, and keeps `TestDropsHangs` out of the run so no test has to reach its timeout.

## To check by hand

- Run the `vstest` tier against a project that skips itself, with `refuse_raw` listing and not listing `dotnet test`, and read the printed line in each case.
  The tests cover the line with a stub, not a real `dotnet` run.
- Run `lym gate test` and `lym gate testmap` from a task worktree's `.scratch` link and read the refusal's way forward.
- Rerun `TestGateTest_DetachedRun` and `TestRedundancyStep` repeatedly, ideally under load, to confirm the flakes are gone.
