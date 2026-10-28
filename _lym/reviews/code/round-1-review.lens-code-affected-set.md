```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, what reads or asserts it, and whether the diff left it wrong:

- **`TestSmokeRemoveLastStrandThenAddRunsTheNewCommand` moved to `smoke_lifecycle_windows_test.go`.** I found one reader, `internal/reedengine/doc.go:485-486`, and it names the old file. Not covered; see F1. `docs/benchmarks/test-redundancy.md` also names it, but it is a measurement report, so I'm not flagging it.
- **`TestLoopLockOwner` moved to `landingdeps_linux_test.go`.** I found no doc or caller that names the old file. The matches in `loomcli/smoke_test.go:4` and `loomcli/landingdeps.go` do not tie it to the old file. Covered.
- **`socketfile_test.go` became `socketfile_unix_test.go`.** The one asserter is the `cmd/lym/tierpurity_test.go` allowlist, and the diff updates it. Covered.
- **`omittedReason` became `isOmitted`, and the tally counts omitted cells.** The old name, `skipWithoutPermissionBits` and `canMakeUnreadable` have no remaining reference. The `livestate_matrix_test.go` comments and test-file messages no longer say "skipped". `fabricengine/doc.go` has no text about skipped matrix cells. Covered.
- **The gitrepo child bodies leave the test runner.** The only remaining mentions of `TestPushCoalescedChildProcess`, `TestLockHolderChildProcess` and `TestBuildLoomShed_OutputShape` are in `docs/benchmarks/test-redundancy.md`. That is a measurement report, so I'm not flagging it. Covered.
- **The verify-gate subtest was deleted, and the decision record promises a board note.** `summary.md` claims a board note `verifygate-misplaced-guard-test` exists. Board writes go through `lym board`, so the diff cannot show it. I raise nothing; the writer's own claim is unverified from the diff.
- **`pattern/PATTERN-test-speed.md` gained a "No test skips by design" section.** `PATTERN.md:60`, the entry's one-line rule, was not changed to carry it. Not covered; see F2.
- **Commit messages and `summary.md`.** No `Pattern-Departure` trailers appear and none is needed. The claims I could check against the diff hold: the `isOmitted` rename, the `TestMain` dispatch, the allowlist move, and the `mustTmuxOutput` break being pre-existing (`switch_integration_test.go` is tagged `tier2 && linux`, and the diff does not touch it). I did not check the `skipAsRoot` claim for `impactset` and `fabricengine`, or the `needsTicks` claim.

**F1 — LOW, consistency.** `internal/reedengine/doc.go:485-486`.
- Summary: the doc names `internal/reedcli/smoke_lifecycle_test.go` as the home of `TestSmokeRemoveLastStrandThenAddRunsTheNewCommand`. The diff moved that test to `internal/reedcli/smoke_lifecycle_windows_test.go`.
- Fix: change the path in that doc comment to the new file.
- Evidence: the diff of `internal/reedcli/smoke_lifecycle_test.go` (the test is removed) and the new `smoke_lifecycle_windows_test.go`.

**F2 — LOW, consistency.** `PATTERN.md:60`.
- Summary: the new no-skip rule lives only in the background file. The `PATTERN-test-speed` entry's rule line does not mention that a test skips only on a missing prerequisite. CLAUDE.md says a new cross-cutting invariant goes into PATTERN in the same commit.
- Fix: add a clause to the `PATTERN-test-speed` entry, such as "a skip names a missing prerequisite, never the OS", keeping the entry's format.
- Evidence: `PATTERN.md:60` against the added section at `pattern/PATTERN-test-speed.md:111-119`.

Tool calls:
1. Grepped `fabricengine` docs for "skipped" text, listed `_lym/landing`, grepped for docs naming moved tests, grepped `loomshed` for the verify-gate note, and read `PATTERN.md`'s test-speed line. My context lacked all of these.
2. Read `_lym/landing/summary.md`, the `reedengine/doc.go` and `loomcli` references, and the matrix comments. My context lacked the summary and the exact reference lines.
