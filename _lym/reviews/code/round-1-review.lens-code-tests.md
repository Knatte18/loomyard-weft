```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (lens code-tests).

**Per behaviour: pinning test**

- **`TestMain` child dispatch, push body.**
  - Pinned by `TestPushCoalesced/separate processes serialize on the lock` (`internal/gitrepo/push_test.go`).
  - If the dispatch broke, the child would run `m.Run` with `-test.run=^$`, run no tests and exit 0.
  - No child would push, so the `remoteBranchSHA == localHead` assertion fails.
  - A child failure (exit 1 plus stderr) reaches the parent through `errs[i] != nil`.
- **`TestMain` child dispatch, lock-holder body.**
  - Pinned by `TestPushCoalesced/a crashed lock holder does not wedge the repo`.
  - A broken dispatch means `LOCK-HELD` never appears, so the 30s `t.Fatal` fires.
  - The `TryAcquireWriteLock` contention check separately pins that the child holds the lock.
- **No `TestPushCoalescedChildProcess` or `TestLockHolderChildProcess` test function.** Pinned only by absence from the source; no test asserts it, which the acceptance criteria don't require.
- **`TestCrossProduct` generating no subtest for omitted cells.**
  - Pinned by `assertCellTally`.
  - `omitted` must equal `len(Omissions) * len(anchors)`.
  - `ran` must equal the per-anchor count of non-omitted cells from `isOmitted`.
  - An omitted cell that ran would break the `ran` count, and an `Omissions` row naming no cell would break the `omitted` count.
- **OS-bound row filters before `t.Run` (D2).**
  - These cover `lifecycle_test.go` `posixOnly`/`windowsOnly`, `rules_test.go`, `archive_test.go` `posixOnly`, `rebaseline_test.go`, `runlevel_test.go`, `watchdog_test.go`, `smoke_lifecycle_test.go`, `fmtverb_test.go`, `configcli_integration_test.go`, `lymcwd_test.go` and `smoke_teardown_test.go`.
  - On Linux each kept row is its own test. The filter conditions are all `== "windows"` or the equivalent.
  - On Linux a wrongly dropped row would be silent, but I read no such inversion.
  - The Windows-side drops cannot be exercised on this host.
- **Rows or tests moved behind `_unix`, `_linux` or `_windows`.**
  - Each moved test is its own pin and keeps its name and tier tag.
  - `GOOS=windows go vet` over the touched packages (tiers 0–2) compiled all that could build without cgo.
  - It reported one error: `internal/reedengine/contract_integration_test.go:842` calls `mustTmuxOutput`, which is defined in `switch_integration_test.go`, a `tier2 && linux` file.
  - Both files are unchanged by this diff, so this is not caused by it.
- **`TestLoopLockOwner` moved to `landingdeps_linux_test.go`, with the skip on missing start time becoming `t.Fatalf`.** Pinned by itself on Linux. A missing start time is now a failure, which matches the "Linux-only" assumption.
- **`TestBuildLoomShed_OutputShape` deleted (D3).** None, but it never ran, so no live coverage was lost. The record's claim that integration-tier loom runs cover the pass-through is not checked by any named test.
- **Always-skipped verify-gate subtest deleted (D6).** No test pins the misplaced-marker findings through the gate key. That was already true while the subtest skipped. The board note the record promises goes through `lym board` into the records repo, so it cannot appear in this diff.
- **`allowedSpawners` entry renamed to `socketfile_unix_test.go`.** Pinned by `cmd/lym/tierpurity_test.go`, which fails if the allowlist key misses the moved file.
- **New `PATTERN-test-speed` section "No test skips by design".** It states "Enforcement is review discipline, not a test" and carries no `(test)` marker, so no test is expected.

**Findings:** none. Every behaviour the diff changes either has a pinning row or had no live coverage before the change.

**Tool calls:** none. My context already held everything I needed.
