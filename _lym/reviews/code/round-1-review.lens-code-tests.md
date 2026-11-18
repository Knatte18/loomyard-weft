```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that would fail if it broke.

**Behaviours and the tests that pin them**

1. **Kill read after the command ends, while the holder keeps the scope:** pinned by `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap/AStartThatReturnsOnceTheCommandHasEnded` (`internal/proc/memorycap_linux_test.go:127-133`). Under the old goroutine watch this row fails every time, because the scope is gone before the read. `AStartThatReturnsAtOnce` pins the ordinary path.
2. **A scope whose `memory.max` is not the cap never runs the command and returns `ErrMemoryCap`:** pinned by `TestRunMemoryCapped_AScopeNotHeldToTheCapIsAnErrorAndNeverRunsTheCommand/AScopeThatIgnoresTheCap` (`memorycap_linux_test.go:210,220`). Without the check, `echo ran` writes output and the marker is written, so `out.Len() != 0` fails and no error comes back.
3. **A scope that never starts is still an `ErrMemoryCap` error and not an `*exec.ExitError`:** pinned by `.../AScopeThatNeverStarts` (`memorycap_linux_test.go:209`).
4. **`wantMax` is the limit rounded down to a whole page:** no test. `testMemoryCap` is `128 << 20` (`memorycap_test.go:17`), a page multiple, so removing or breaking the rounding fails nothing.
5. **The holder points its stdio at `/dev/null`, so a wait for the command's output ends:** pinned by every scope test that sets `cmd.Stdout = &out`, such as the Via rows (`memorycap_linux_test.go:142-143`) and `AKillAtTheCap` (`:66-67`). A holder on the stdout pipe stalls `cmd.Wait` until `release` closes, and `release` closes only after the wait. That deadlocks, and the test times out.
6. **The holder ends on every return when `release` closes:** no test. No test checks that the holder process, or the scope it keeps, is gone after a run with no kill or no cancel, such as `TestRunMemoryCapped/AChildWithinTheCapEndsOnItsOwn` (`memorycap_test.go:48`). After a kill at the cap, `killScope` ends the holder, so the rows with a kill do not show this either.
7. **The holder leaves the shell's tree, so the command never has it as a child:** no test. None of the test commands waits for its children.
8. **The holder is closed off fds 3 and 4 and the command gets neither:** there is no direct test, but breaking it does no observable harm. A leaked fd 3 only delays the bounded marker read. No finding.

The redundancy report lists `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap` as "not judged, out of process", so it is not a contained candidate and needs no `//testtiming:keep` mark. Each new row covers behaviour that no other row covers, so I found no duplication.

**Findings**

- **MEDIUM | design | `internal/proc/memorycap_linux.go:161-162`** (the `release` defer) and `scopeStartScript` at `:62`. Nothing tests that the scope holder ends once the call returns. If `release` were never closed, or the holder ignored end of file on fd 4, every capped run with no kill would leave a process and its scope behind, and every test would still pass. Evidence: `memorycap_test.go:48-58` and `memorycap_linux_test.go:79-106` assert only the returned values and the sleep's pid, never the holder or the scope's cgroup. A possible fix: one row whose command prints its cgroup (`cat /proc/self/cgroup`), then a bounded poll until `/sys/fs/cgroup/<scope>` is gone once the call returns.
- **LOW | design | `internal/proc/memorycap_linux.go:163-164`.** Nothing tests rounding `wantMax` down to a whole page. Every test uses `testMemoryCap = 128 << 20` (`memorycap_test.go:17`), which is already page-aligned, so a cap that is not a page multiple goes unexercised. If the rounding broke, a legitimate cap would be refused with `ErrMemoryCap`. A possible fix: a row in an existing scope test with a cap that is not a page multiple, such as `testMemoryCap + 1`.
- **LOW | design | `internal/proc/memorycap_linux.go:53,62`.** Nothing tests that the holder leaves the shell's tree. The `( ... & )` subshell exists so the command never sees the holder as a child of its own, and no row would fail if the holder were started as a direct background child of the shell that becomes the command. Evidence: none of the test commands at `memorycap_linux_test.go:64,89,140,164,234` waits on its children.

I made no tool calls; my inherited context covered everything this lens needed.
