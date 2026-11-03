```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

1. `taskWorktreePresent` reads a stray path as absent (plain file, empty dir, ignored-output dir) and a real worktree as present. It is pinned by `TaskWorktreePresentReadsStrayPathsAsAbsent` (`readopt_integration_test.go:~270-295`). A revert to `return false, err` fails the stray rows. The present row is pinned by `present-pair`.

2. The re-adoption `Gone` seam is now the negation of `taskWorktreePresent`. It is pinned by `TestBattenIntegration_GoneTaskWorktree`, whose re-adoption rows assert the worktree is present afterwards. They also drive the re-make from absent directories, which exercises `Gone`.

3. A teardown resume over a stray directory judges the owner on the local board. It is pinned by `TeardownResumeOverAStrayDirectoryJudgesTheLocalBoard`. Before the change, `ownerPreflight` returned the resolver error and the test would fail. One gap: it calls `ownerPreflight` on a receiver from `readoptReceiver`. The test asserts only nil, so it would also pass if the owner were judged elsewhere. That is acceptable.

4. `Promoted` returns false for a slug the board no longer holds, or holds as a non-task. It is pinned only at the integration tier, by `MarkedPairTheBoardNoLongerHoldsResumes` (the `landed` slug, absent from the board). The `promoted` slug pins the task case, and the project case was already covered by the existing project rows. The unit table in `startguard_test.go` stubs `Promoted`, so it does not reach this logic. The integration pair covers it, so I raise no finding.

5. The `ReachedParent` seam passes a marked pair whose work reached its parent. It is pinned by the unit row `MarkedPairTaskWhoseWorkReachedItsParentPasses`. The row for the opposite case, a marked pair whose work has not reached its parent, still refuses. It is pinned by the existing "carries the beam mark" rows, which leave `reached` false.

6. A `ReachedParent` probe error refuses, naming the probe and the re-run. It is pinned by `ReachedParentProbeErrorRefusesNamingTheProbeAndTheRerun`.

7. `hubStartGuard` wires `ReachedParent: hubChildReachedParent(...)`. It is pinned only by the `promoted` integration subtest, which exercises the real wiring with no child status and reads not reached. No test shows the guard passing a marked task whose work really reached its parent through the real probe. `MarkedPairTaskWhoseWorkReachedItsParentPasses` stubs the seam. The `hubChildReachedParent` extraction is covered through the unchanged cancel tests. If the wiring were dropped, the guard would reach a nil `ReachedParent` and keep refusing a task whose work reached its parent. No test would fail, because the unit tests inject their own seam and the integration test has no reached case. This is a gap, but a small one. A reached case with a Worktree-Teardown batten status over the real wiring needs only a status file. Severity LOW, class `scope`. Location: `internal/battencli/beamland_integration_test.go:~107-120`.

8. `hubChildReachedParent` extracted from `hubCancelDeps` is a behaviour-preserving move. The existing cancel tests cover it, and I found no new gap.

9. Removed assertion: `taskWorktreeGone` on an absent path in `wire_test.go`. The behaviour moved to `taskWorktreePresent`, whose absent case is still asserted in the same test (`never-created` is false) and in the integration rows. Nothing is left uncovered.

10. Doc-only changes (`cli.go` package doc, `wire.go` comments) are exempt as static prose.

**Findings**

- LOW, `scope`: no test drives `hubStartGuard`'s real `ReachedParent` wiring through the reached case. The unit row stubs the seam, and the integration subtests cover only the not-reached case (`promoted`) and the board-absent case (`landed`). Location: `internal/battencli/beamland_integration_test.go:107-120`. Evidence: `startguard_test.go:233-240` (stubbed seam) and `beamland_integration_test.go:107-120`. Fix: add an integration subtest that gives the `promoted` slug a batten status at Worktree-Teardown and asserts `refuse` returns nil.

No test-economy defects: I found no duplicate tests, no per-helper tests that the public surface already covers, and no assertions on prose.

**Tool calls:** 0. My inherited context already held every diff hunk and file I needed.
