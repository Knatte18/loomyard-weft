# Agent prompt gives no way to run slow golden tests by hand; gate skips them silently

`gate-skipped-tests-silent` · Note · bug, gate, dotnet, stencil

Imported from [issue #127](https://github.com/Knatte18/loomyard/issues/127).

## What happened

A task agent had to run a project's slow test suite by hand, because the gate's `COMMONLIB_REPO` setting makes the gate skip those tests.
`dotnet test` on the slow-test project printed only restore lines and exited 0: no test ran and no message said why.
The project disables itself as a test project unless an opt-in MSBuild property is passed, so the run looked like a pass.
Finding the property took several runs.
Neither the agent's prompt nor the harness README mentions it.

## Why it is abnormal

A gate that skips a test class, plus a by-hand run that exits 0 without running anything, lets an agent believe tests passed when none ran.
The agent has no lym-provided signal that the gate skipped them or how to run them.

## What to do

- Have the gate report which test classes it skipped and why, as a visible line in its output.
- Have the agent prompt, or the harness README, tell agents how to run gate-skipped tests by hand.
- Consider a lym check that flags a `dotnet test` run that executed zero tests.

Provenance: task `td-transient-odd-even-chatter`; note `Darn-Write.md`.
