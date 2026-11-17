```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens, finding for each added or changed behaviour the test that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **`-p OOMPolicy=continue` keeps a kill at the cap from failing the scope with `oom-kill`, so the desktop raises no notice.** No test pins this. Removing the property leaves every test passing:
   - `TestRunMemoryCapped_EndsAChildOverTheCapByTheHostsMeans/AScopeReportsTheKernelsKill` still sees SIGKILL.
   - `TestRunMemoryCapped_AKillAtTheCapEndsWhatTheChildLeftInItsScope` still passes, because under the default `OOMPolicy=stop` systemd stops the scope and kills the sleep itself.
2. **`killed` stays true for a SIGKILL at the cap under the new policy.** Pinned by `AScopeReportsTheKernelsKill` (`memorycap_linux_test.go:36`) and by the new test (`memorycap_linux_test.go:67`).
3. **The start script writes the scope's cgroup listing and `scopeCgroup` parses it.** Pinned on the positive path: a broken parse returns `ok` false, so `killed` comes back false and `AScopeReportsTheKernelsKill` fails. The no-listing path is pinned by `TestRunMemoryCapped_AScopeThatNeverStartsIsAnErrorAndNotTheCommandsExit` (`memorycap_linux_test.go:90`).
4. **`killScope` ends what the killed command left in its scope.** Pinned by `TestRunMemoryCapped_AKillAtTheCapEndsWhatTheChildLeftInItsScope`: under `OOMPolicy=continue`, `sleep 300` outlives the call without the kill, so the poll fails. A zombie or a recycled pid keeps `/proc/<pid>` present, so either can only fail the test falsely, never pass it falsely.
5. **`killScope` treats `ErrNotExist` and `ENODEV` as no error, and joins any other failure into `err`.** No test pins this. Every SIGKILL path already returns a non-nil `*exec.ExitError`, so the existing rows' assertions (`killed && err != nil`) hold whether the kill error is joined or not.
6. **Cancel, non-SIGKILL and data-limit paths skip `killScope`.** These are unchanged behaviour, already pinned by `TestRunMemoryCapped_ACancelEndsTheRunAndIsNoCapKill`, `TestRunMemoryCapped/AChildWithinTheCapEndsOnItsOwn` and the `WithNoUserManagerTheChildFailsItsOwnAllocation` row.

**Findings:**

- **MEDIUM, design.** `internal/proc/memorycap_linux.go:101` (`memoryScopeArgs`, `OOMPolicy=continue`). The behaviour the task exists for has no test that fails if it breaks: a kill at the cap leaving the scope's result something other than `oom-kill`. The new test passes with or without the property, because the default `stop` policy also kills the leftover sleep. Evidence: `memorycap_linux_test.go:53-83` asserts only `killed`/`err` and that the pid is gone; no row reads the unit's result. A test could keep the scope from being collected, or watch the unit's `Result`, and assert it is not `oom-kill`.
- **LOW, design.** `internal/proc/memorycap_linux.go:180-187` (`killScope`). The tolerance of a vanished scope (`ErrNotExist`/`ENODEV`) and the joining of any other failure into `err` are unpinned. The joined error changes only the error's text, since `err` is already a non-nil `*exec.ExitError` on this path (`memorycap_linux.go:156-162`), so no existing assertion can tell the two apart. Evidence: the assertions at `memorycap_linux_test.go:46` and `:67` check only `err == nil` and `killed`.
- **LOW, consistency.** `internal/proc/memorycap_linux_test.go:75-82`. The poll compares a measured real time against a 10 s bound to decide whether the sleep ended. PATTERN-test-speed's "Time is injectable" section says a deadline used this way is a generous minute (`pattern/PATTERN-test-speed.md:178-180`). The existing cancel test in the same file uses a minute (`memorycap_linux_test.go:128`).

Tool calls: none; every file the lens needed was already in the inherited context.
