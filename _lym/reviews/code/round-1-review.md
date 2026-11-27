---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/drive.go:535
    summary: The primeShuttle change that keeps an absent task worktree quiet before Worktree-Create, the fix of the issue's false stop premise, has no test that fails if it breaks.
    fix: "Add a tier0 test, or a row of `TestPrimeShuttle_WiresTheReplyProbe` in internal/battencli/wire_seams_test.go, that builds `primeShuttle` over a prime with no task worktree for a non-empty slug and asserts the trace holds no `locate the task worktree to record its tackler and arbiter sessions failed` warning. Use the logger's existing trace output as the observable. If no test can read that output, move the present-and-locate decision into one function in drive.go that returns the task location, or nil, and the error to log, call it from primeShuttle, and test that its absent-pair case returns nil, nil and its stat-error case returns the error."
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/fabricengine/weftbranch.go:41
    summary: The board entry asks Worktree-Create to retry a network failure of a remote lookup, but only Add's two pre-flight probes retry. The origin lookup and fetch in resolveWeftBranch, the adopted-warp and weft fast-forward fetches in Add, and the TaskBranchOnOrigin lookup in beamForkParent still fail on the first transport failure.
    fix: "Wrap the `remoteHeadTip` call and the `FetchWithArgs` call in `resolveWeftBranch` (internal/fabricengine/weftbranch.go:41-56) in the same transport retry as the probes. Do the same for Add's adopted-warp fetch (internal/fabricengine/add.go:526) and its weft fast-forward fetches (add.go:596 and :604), and for the `TaskBranchOnOrigin` closure passed to `beamForkParent` in internal/battencli/wire.go:1179. If a site stays unwrapped, say in _lym/landing/summary.md's `## What changes` which lookups retry and which do not."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: _lym/landing/summary.md:24
    summary: The change description says a retry extends the prime-lock hold by at most seven seconds. A timed-out probe also counts as a transport failure, so each of the three re-runs can add a full remote deadline on top of the seven seconds of backoff.
    fix: "In _lym/landing/summary.md, replace `a retry extends the time Add holds the hub's prime lock by at most seven seconds` with a sentence saying a retry adds seven seconds of backoff plus up to three more attempts of each probe, each bounded by git's remote deadline."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/add_remoteleftover_integration_test.go:117
    summary: No row pins that a probe failure which is not a transport failure is returned at once with no backoff, so a regression that retries every error passes every test.
    fix: "Add a row to `TestAdd_RetriesOriginProbesThatFailInTransit`, such as `OriginNotARepository`, that sets both stores' origin to a missing local path, which git refuses with `does not appear to be a git repository`. The row asserts that Add returns an error for which `gitexec.IsTransportFailure` is false, that `sleeps.Load()` equals its `wantSleeps` of 0, and that `requireNothingCreated` holds. Add a sentence to the test's doc comment naming this case."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/fabricengine/add_remoteleftover_integration_test.go:146
    summary: The BackoffSleep hook calls gitkit.Git from Add's probe goroutines. gitkit.Git calls Fatalf when git fails, and FailNow is forbidden off the test goroutine, so a failed restore misreports instead of failing cleanly.
    fix: "Inside the hook's `restore.Do`, run each `git remote set-url` through a call that returns its error, not one that fails the test, and keep the first error in a variable the closure captures. After `Add` returns, on the test goroutine, call `t.Fatalf` with that error when it is set."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/fabricengine/add_remoteleftover_integration_test.go:122
    summary: The wantSleeps value 0 in the OriginAnswersAgain row is never read, because that row checks sleeps against 0 inline, so the table field reads as an assertion it is not.
    fix: "Set the `OriginAnswersAgain` row's `wantSleeps` to 1, and in its branch replace `sleeps.Load() == 0` with `sleeps.Load() < tt.wantSleeps`. Reword its message to say at least `tt.wantSleeps` backoff waits were wanted."
---

### [MEDIUM] The quiet absent-pair path of primeShuttle has no test

**Location:** `internal/battencli/drive.go:535`

**Issue:** The issue's false stop premise came from the warning `primeShuttle` logged with `absentTaskWorktreeRefusal`'s text, "this run's durable status says it was already created", when the tackler or arbiter was built before Worktree-Create had made the pair.
The diff now checks `taskWorktreePresent` first and stays quiet when the pair is absent.
`TestPrimeShuttle_WiresTheReplyProbe` (`internal/battencli/wire_seams_test.go:302`) runs with no task worktree, but it asserts only `HasReplyProbe`, so it passes with and without the change.
No test would catch a regression that brings the warning back, or one that stops wiring the usage recorder for a present pair.

### [LOW] Worktree-Create's other remote lookups do not retry

**Location:** `internal/fabricengine/weftbranch.go:41`

**Issue:** The goal reads "Worktree-Create retries a network failure of a remote lookup a few times with backoff before it blocks."
The diff retries only Add's warp and weft pre-flight probes (`add.go:288-307`), which covers the issue's reported `ls-remote` of the weft branch on a fresh task.
Worktree-Create still reaches these remote calls with no retry:
- `resolveWeftBranch`'s `remoteHeadTip` and fetch (`weftbranch.go:41`, `:54`), on a live weft that is only on origin;
- Add's adopted-warp fetch (`add.go:526`) and weft fast-forward fetches (`add.go:596`, `:604`);
- `beamForkParent`'s `TaskBranchOnOrigin` (`internal/battencli/wire.go:1179`), for a beam member whose beam pair is absent.

A transport failure in any of them still stops the row on its first attempt.
The fresh-task path the issue reports runs none of them.

### [LOW] The change description understates how long a retry holds the prime lock

**Location:** `_lym/landing/summary.md:24`

**Issue:** `IsTransportFailure` counts a `*GitError` with a non-zero `Timeout` as a transport failure (`internal/gitexec/transport.go:84`).
A probe that times out is therefore run again three times, and each attempt can last up to git's remote deadline.
All of that runs under the hold that Add takes before it waits for the probes (`add.go:311-316`, `:472`).
The description's "at most seven seconds" counts only the backoff sleeps.

### [LOW] No row pins that a non-transport failure is not retried

**Location:** `internal/fabricengine/add_remoteleftover_integration_test.go:117`

**Issue:** The test's only failure is `connection refused`.
The early return at `add.go:95` for an error that is not a transport failure has no row with a sleep count.
The existing broken-origin tests leave `BackoffSleep` nil and count nothing.

### [LOW] The test's backoff hook can call Fatalf off the test goroutine

**Location:** `internal/fabricengine/add_remoteleftover_integration_test.go:146`

**Issue:** `BackoffSleep` runs inside Add's probe goroutines (`add.go:290-306`).
The hook's `gitkit.Git(t, …)` calls `tb.Fatalf` when git fails (`internal/gitkit/gitkit.go:64`).
The testing package forbids calling `FailNow` from any goroutine other than the test's own.

### [NIT] An unread table value in the retry test

**Location:** `internal/fabricengine/add_remoteleftover_integration_test.go:122`

**Issue:** `{"OriginAnswersAgain", true, 0}` sets `wantSleeps`, but that row's branch returns before reading it and checks `sleeps.Load() == 0` inline.

## Rejected

- `originProbeBackoffDelays` as a package-level `var` against PATTERN-test-speed's "Time is injectable": the wait goes through the `AddOptions.BackoffSleep` struct field, so no test waits out the delays or has to set the var.
- The added latency of a refused Add that waits for retrying probes (`add.go:309`): Add's doc already says every return waits for the probes, and a backoff bounded by the delays is the design the goal asks for. Only the description's figure is kept, as F3.
- The new recipe sentences' "never a pair to restore or abandon": each applies only to an absent task worktree and an absent records branch. A partly made pair (`taskWorktreeComplete`'s present case, `wire.go:1162-1174`) and a leftover branch (`createRefusal`'s `ErrBranchExists` and `ErrRemoteLeftover` cases) are present parts with their own ways forward, which the text does not cover.
- Another path handing the tackler or arbiter the absent-worktree refusal at Worktree-Create: `battenPreGoto` reads the child only for Land and Worktree-Teardown targets (`goto.go:36-38`), `readChildStatus` swallows the error (`arm.go:1109-1113`), and `readNoticeChild` and `ReleaseRunParentLock` check presence first.
