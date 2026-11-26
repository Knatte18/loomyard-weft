# gate reach: functions called directly by tier1 tests are reported as reached by no mapped test

`reach-misses-tier1-calls` · Task · bug, gate, testmap · starting · Open-Window

Depends on: [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md)

Imported from [issue #250](https://github.com/Knatte18/loomyard/issues/250).

## What happened

During a code review fix round, the reach note listed `hubCancelDeps`, `Topology.SettleGonePairTeardown` and `Topology.RemoveLocal` as reached by no mapped test.
Each is called directly by a tier1 test.
The maps of both packages belonged to the worktree itself and were minutes old.
`lym gate reach` gave the same answer.
The note carried the caveat "may run it unseen".

## Why it is abnormal

A function that a tier1 test calls directly should be mapped to that test, above all with a fresh map of the worktree's own.
The false "unreached" answer could only be ruled out by reading the tests and running them.
The caveat does not say why the map missed these tests, so the reader cannot tell a map defect from a real coverage gap.

## What to do

- Find why the reach map misses direct calls from tier1 tests, and fix the mapping.
- Where the map cannot see a call, have the caveat name the reason for that function.

Provenance: task `remove-record-rerun-pair-gone`, note `burler-code-r2.md`.
