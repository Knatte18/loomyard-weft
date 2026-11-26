---
round: 1
focus:
  - "internal/loomcli/smoke_helpers_test.go, waitForFixtureProcessesToExit cleanup loop: trace the states after the first kill, including a pid that first appears under the root after `killed` is set and a pid that exits and is respawned. Settle whether every process left under the root is ended and whether the failure message and the doc comment match what the loop does."
  - "Fixture coverage against the board entry's `every lym board process it starts`: enumerate the fixtures and tests in internal/loomcli whose verbs can spawn a detached `lym board sync` (including all users of newWiredPairFixture and newBadReedUpFixture, in every tier), and settle whether each one ends it before the test returns."
  - "Whether the scan can see the processes it is meant to end: check that the spawned sync and git processes have a cwd under filepath.Dir(t.TempDir()) of the test or subtest that registers the cleanup, including for hubforge copies and subtests."
  - "Cleanup order: for each caller, settle whether the wait-and-kill runs after any teardown that stops the fixture's own processes and before the temp-dir removal, and whether it can now kill a process another cleanup still expects to stop itself."
  - "internal/loomcli/fixtureprocesses_integration_linux_test.go: settle whether the test fails if the kill is removed, whether its file name and tier gate follow the repository's test rules, and, for the redundancy report's contained listing, which of its assertions the covering test also holds."
  - "_lym/landing/summary.md: check its claims against the code, in particular the failure mode and tier it names, the symlink explanation, and which tests the new fixture registration covers."
---

Round 1 seed.
The diff is test-only code in `internal/loomcli`, so the weight falls on whether the cleanup ends every process the board entry names,
and on whether its new kill path is correct.
