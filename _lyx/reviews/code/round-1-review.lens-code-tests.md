```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it.

**Behaviour → pinning test**
- Config load (`{baseline_dir}` required, `timeout_min` at least 1, undeclared step refused naming `lyx config drift`): `TestLoadConfig` in `config_test.go`. `Step` refusal: `baseline_test.go:368`.
- Counted-capture predicate: `TestCountedCapture` (7 rows, `records_test.go:24-30`).
- Run counter (`TakeRun`, `ResetRuns`, `AttemptHead`, `CloseAgentSession`): `TestTakeRun`, plus `verify_test.go:319-338` (reset at a fresh spawn).
- Capture run (pass rule, no file written, non-zero exit, timeout tree kill, numbered backup, lock-retry window, slot wait, `Check` refusal, side capture): `TestRunCapture` (`capture_test.go:99-257`).
- Main-capture precondition: `TestCheckMainCapture`.
- Compare run (marker held while queued, held marker refused, timeout, skip list rendered): `TestRunCompare`.
- `SkipList`: `TestSkipList`.
- `StartCapture` (pid after the record, failed starts, held lock, breakaway refusal and fallback, `ProbeLock` on a missing directory): `TestStartCapture`.
- Capture outlives the caller, teardown ends it, `lyx drift compare` refuses past the budget: `TestCapture_OutlivesCaller` (`capture_integration_test.go`).
- Verbs:
  - `RecordScope`: `TestRecordScope`.
  - `Accept`: `TestAcceptWithdraw`, "Accept" rows `verbs_test.go:135-146`.
  - `Withdraw`: `TestAcceptWithdraw`, "Withdraw" subtests `verbs_test.go:175-208`.
  - `Defer` and `Waive`: `TestDeferWaive`.
  - `CheckVerifyRow`: `verbs_test.go:217`.
- `Drift-Baseline` re-entry cases (scope without `baseline.md`, `awaited_at` set, running record with a free lock, failed record, moved HEAD, dirty tree, breakaway fallback, held lock waited on in darn and left running in loom, agent recording no scope): `TestDriftBaseline` (`baseline_test.go:183-376`).
- `Drift-Await` cases (no record, running with a free lock, failed, timed out, lock released as the record flips, HEAD moved, `awaited_at`, after `Defer`): `TestDriftAwait` (`await_test.go:34-136`).
- `Drift-Verify` (the first four rows are each a subtest of `TestDriftVerify`):
  - Done paths: `verify_test.go:243`.
  - No counted capture as a decision halt under every scope, in loom and darn: `:265`.
  - Undeclared compare: `:300`.
  - Failing compare spawns one agent: `:319`.
  - Final compare still failing, prefix by scope: `:351`.
  - Dirty tree re-prompt: `:370`.
  - Verify entry failing its attempts: `:398`.
  - Fix after a re-prompt compared again: `:419`.
  - Records-side dirt is no dirty tree: `:444`.
  - Gate-halt resume: `:459-574`.
  - Interrupted agent: `:595-645`.
  - Scope `full` ruling: `:661-736`.
- Activity reading (held capture lock, held compare marker, dead lock reads as none): `TestAgentActivity_LiveLockWaitReadsAsActivity` rows `reviewstatus_test.go` (diff lines 54-66).
- Recipe `entry_gate` on all three rows: `TestDriftRows_EntryGate`; registry entries: `TestDriftEntries`; stencil tables: `TestSeatTables`.
- Teardown ordering of `killCapture` in `EndSession`: `TestTeardown_CallSequence` (`teardown_test.go:243-321`), with `killCaptureErr` propagation.
- Windows `DetachBreakaway` flags: `proc_windows_test.go:53`.

**Findings**

1. LOW, design. Location: `internal/proc/namedjob_windows.go:28`. No test pins that the named job and the `StartTied` job carry `JOB_OBJECT_LIMIT_BREAKAWAY_OK`. The only breakaway test is the `DetachBreakaway` flag test at `proc_windows_test.go:53`. The plan accepts this because the limit runs only on Windows, so I record it as a stated gap rather than a defect. Evidence: the grep in `proc` found `StartTied` tests only in `tied_linux_test.go`.
   - Fix: add a Windows tier test that creates the job and asserts the limit flags through `QueryInformationJobObject`, or accept the gap under the plan's Windows decision.

2. LOW, design. Location: `internal/driftcli/cli.go:403`, where the capture child calls `proc.HoldNamedJob(driftengine.CaptureJobName(...))`. No test found that the child holds the job, and the capture teardown test fakes `killCapture` (`teardown_test.go:203`). `CaptureJobName` has no test either. This is Windows-only and the plan accepts it.
   - Fix: add a tier0 test that `CaptureJobName` is stable per worktree root and equals the name `pairteardown` computes.

3. Not a defect (Test economy): `TestSkipList` in `skiplist_test.go` tests the exported `SkipList`, which `RunCompare` and the producers consume. No existing test covers the withdrawn or duplicate rows, so it is not redundant.

No redundant test found. No per-helper test that a public-surface test already covers. No review-fix test added for a non-gap finding. No new assertion on a stencil's static prose was visible in the tests I listed.

**Not verified**
I did not open the `driftcli` `status`, `capture` and `waive` verb tests. `driftcli` has only `capture_integration_test.go`, so the CLI glue in `cli.go` (flag wiring, `lyx drift status` output) is pinned only by that file's `lyxRun` and `statusBody` helpers. I did not confirm that `lyx drift compare` and `lyx drift capture --side` refuse outside `Drift-Verify`. `cli.go:184` shows `CheckVerifyRow` is called, and the engine-level `CheckVerifyRow` test covers the rule. The integration test refuses `compare` at `capture_integration_test.go:208`.

**Tool calls**
1. Listed every test function in the drift test files — context lacked the test inventory.
2. Grepped `verify_test.go` subtest names, plus `agentActivityFor`, `ProbeLock` and `StartTied` coverage — context lacked the test bodies.
3. Listed the subtest names of the other drift test files — same reason.
4. Read the `reviewstatus_test.go` drift diff, the `driftcli` test layout, the `TakeRun` and `ErrRunsSpent` usage, `CaptureJobName` test usage, and `skiplist_test.go`.
