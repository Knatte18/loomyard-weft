```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes on a concrete input.

**Cases run**

- **`runLoop`, directory creation fails** (`internal/shedverbs/loop.go:213-215`). `endWithoutEnvelope` logs the reason, and `appendLoopLogMark` tries to write the mark. The loop log sits in the same steps directory, so `appendLoopLog`'s own `MkdirAll` also fails; it warns and returns, and the function returns 1. The waiter wrote its starting mark into that directory before it started the keeper, so the directory exists in practice. Correct.
- **Lock times out with `err == nil, locked == false`** (`loop.go:217-222`). The reason has no error suffix, and the mark is keyed by the second loop's own id. `loopLogLinesOf` (`loopwait.go:272-280`) skips another loop's mark as neither mark nor output, so the owning loop's evidence is unaffected. Correct.
- **Lock fails with `err != nil`.** The reason ends `: <err>`. Correct.
- **Refusal on a dead loop's pid file** (`loop.go:225-227`). The mark is written while the lock is held, and the function returns 1. Correct.
- **Pid-file write fails** (`loop.go:243-245`). No pid file exists, the mark is written, and the function returns 1. The waiter's early-exit stop reads the mark. Correct.
- **`StartedAt` through the in-flight updates.** `stepUntilStop` rewrites `*pid` (`loop.go:413-414`), and the copy keeps `StartedAt`. Pair teardown restores the record it read back (`pairteardown/loop.go:80-92`), which also keeps it. An older file without `started_at` decodes to zero. A JSON round trip drops the monotonic reading, and `Equal` still holds. Correct.
- **Envelope write fails** (`loop.go:356-358`). The mark is written and the pid file stays. The next waiter's `deadLoopStop` lists the mark in its first clause and skips `unmarkedExitCause`. Correct.
- **Panic in `OnStart` or `OnExit`.** It is not recovered and adds no new mark; the keeper's exit-status mark and the crash excerpt still cover it. The signal path after stepping keeps its existing signal mark. Correct.
- **`endWithoutEnvelope`**: all callers pass a non-empty reason. Correct.
- **`loopExitEvidence` with marks, without a start mark, and unmarked.** Only the unmarked branch calls `unmarkedExitCause`. `loopExitedEarlyStop` passes a zero time and gets the old wording. Correct.
- **`unmarkedExitCause` with zero `startedAt`, or `BootTime` false**: old wording. Correct.
- **`unmarkedExitCause` with boot equal to the start**: `After` is false, so it names a kill. The whole-second truncation of `btime` only matters for a reboot within a second of the loop's start, which is negligible. Correct.
- **`unmarkedExitCause` when the wall clock steps forward after the loop starts**: wrong result, finding 1.
- **`unmarkedExitCause` on Windows across a shutdown with Fast Startup**: wrong result, finding 1.
- **`proc.BootTime` on Linux.** An unreadable file, a missing `btime` line and a `btime` that fails to parse each return false; `btime 1700000000` returns `time.Unix(1700000000, 0)`. Correct.
- **`proc.BootTime` on Windows.** It returns the wall clock less `GetTickCount64`, so it shifts with any wall-clock step; this is part of finding 1.
- **`loopAgreesWithState`.** Held beside busy with cause `background` now agrees. Held beside busy with cause `turn` still disagrees, so a stuck turn is still logged. Held beside idle-stalled is unchanged. Correct.
- **Test clock seams.** In the panic tests, the first read returns real time and the second panics after the child is recorded, which matches the new extra read at loop start. Correct.

**Findings**

1. **MEDIUM, design** — `internal/shedverbs/loopwait.go:229-249` (`unmarkedExitCause`), with `internal/proc/proc_linux.go:52-70` and `internal/proc/proc_windows.go:79-83`.
   - **Summary:** The boot-time inference is not robust to wall-clock changes, yet it prints a definite cause, so the halt note can name the wrong one.
   - **Linux:** `/proc/stat`'s `btime` is the current real time less the time since boot. A clock step after the loop starts (NTP or chrony `makestep`, WSL2 or VM resync after host sleep) moves `btime` forward by the step. Once the step exceeds the time between boot and the loop's start, `boot.After(startedAt)` holds and the note says "a stop of the machine ended it", though the machine never stopped.
   - **Windows:** `time.Now() - GetTickCount64` shifts with a clock step in the same way. With Fast Startup, a shutdown and power-on does not reset the tick count, so a machine stop reads as "a kill … as by SIGKILL or the out-of-memory killer". That wording is also Linux-specific.
   - **Evidence:** `loopwait.go:234-241` (the `After` comparison and the definite wording), `proc_linux.go:67` (`btime`), `proc_windows.go:80` (`time.Now().Add(-windows.DurationSinceBoot())`).
   - **Fix:** Record an identity of the boot that cannot step. On Linux that is `/proc/sys/kernel/random/boot_id`, stored in the pid record and compared at stop time. Otherwise, compare against a tolerance and hedge the wording ("likely") so the note never asserts a cause the evidence cannot carry.

The code-correctness lens finds no other defect.

Tool calls:
1. `grep` of `internal/shedrun/paths.go` and of `appendLoopLog` in `loop.go`: to check whether the loop log shares the lock's directory, which bears on the directory-creation branch. My context had neither path nor `appendLoopLog`'s body.
