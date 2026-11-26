```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test that fails when it breaks.

**Per behaviour, the test that pins it**

1. `waitForFixtureProcessesToExit` kills a process under the root that outlives its patience (`smoke_helpers_test.go:138-144`).
   - Pinned by `TestWaitForFixtureProcessesToExit_KillsAProcessThatOutlivesItsPatience`, which asserts SIGKILL (`fixtureprocesses_integration_linux_test.go:51-54`).
   - Removing the kill fails it: after the second 100 ms patience the subtest's `t.Errorf` fires, and without the cleanup the `select` times out at 5 s.
2. The cleanup returns without killing when no process is left under the root (`smoke_helpers_test.go:126-129`).
   - No test fails if this breaks; a cleanup that always kills would still pass every test.
   - It shows only as fewer kill logs, so it is not filed.
3. The cleanup fails the test when a killed process still runs after a second patience (`smoke_helpers_test.go:134-137`).
   - None. A process that survives SIGKILL cannot be built in a test, so it is not filed.
4. The temp root is resolved through `filepath.EvalSymlinks` before the scan (`smoke_helpers_test.go:112-115`).
   - None. The new test's root sits under the `TMPDIR` that `tmuxkit.Main` sets.
   - Nothing makes that path hold a symlink, so the test passes with or without the resolution.
5. `newBadReedUpFixture` registers the cleanup itself, which covers every subtest of `TestLoomFailedReedUpRefuses` (`smoke_helpers_test.go:259`).
   - None that is deterministic. `tmuxkit.Main`'s end-of-package sweep catches a leak only when the sync outlives the package, which depends on timing.
6. The kill is logged with the process's argv (`smoke_helpers_test.go:139-140`).
   - None. It is a log line, so no finding.
7. The `patience` parameter is honoured (`smoke_helpers_test.go:109`).
   - Pinned by the new test's 100 ms value: with a 10 s patience, the 5 s `select` would fail.
8. The scan reaches a process that first appears under the root after `killed` is set, such as a git child of a killed sync.
   - None. The code-correctness lens owns whether the behaviour is right.

**Findings**

- **LOW · design** · `internal/loomcli/smoke_helpers_test.go:112`
  - The `EvalSymlinks` resolution of the temp root has no test that fails without it. The summary names it as part of the fix.
  - Evidence: the diff at `smoke_helpers_test.go:112-115`. The new test's root comes from `t.TempDir()` under `tmuxkit.Main`'s `TMPDIR` (`fixtureprocesses_integration_linux_test.go:33-35`), and nothing symlinks it.
- **LOW · design** · `internal/loomcli/fixtureprocesses_integration_linux_test.go:26`
  - The redundancy report lists the new test as contained by `TestAgentActivity_LiveLockWaitReadsAsActivity`, which asserts nothing about process cleanup.
  - The new test pins a side effect coverage cannot see: a SIGKILL on a process under the temp root, done by test-only code that coverage does not instrument.
  - Fix: mark it `//testtiming:keep` directly above its `func Test…` line, with a reason naming that it pins the kill of a process left under the fixture's temp root.
  - Evidence: the redundancy report in the rubric; `reviewstatus_test.go:407-414`; the assertions at `fixtureprocesses_integration_linux_test.go:45-58`.
- **LOW · design** · `internal/loomcli/smoke_helpers_test.go:259`
  - Nothing pins that `newBadReedUpFixture` registers the cleanup. Dropping it brings back the reported intermittent leak, and only the timing-dependent end-of-package sweep could catch it.
  - A deterministic check would need a fake sync, and the record's "No test seam" decision accepts this gap.
  - Evidence: the diff at `smoke_helpers_test.go:256-259`; `internal/testkit/tmuxkit/tmuxkit.go:180`; `decision-record.md:27-31`.

Tool calls: none; my context already held everything the procedure needed.
