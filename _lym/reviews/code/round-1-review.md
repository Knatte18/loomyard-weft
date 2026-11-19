---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loopwait.go#unmarkedExitCause
    summary: The cause of an unmarked exit is inferred from a wall-clock boot time and stated as fact, so a clock step after the loop starts, or a Windows Fast Startup shutdown, names the wrong cause.
    fix: "Replace the wall-clock comparison with a boot identity that does not move with the clock. Add proc.BootID in internal/proc (Linux: read and trim /proc/sys/kernel/random/boot_id, reporting false when unreadable; Windows: always \"\", false) and document it in internal/proc/doc.go, replacing proc.BootTime and its doc paragraph. In internal/shedverbs/looppid.go replace LoopPIDRecord.StartedAt with BootID string `json:\"boot_id,omitempty\"`, which runLoop fills from proc.BootID before its first WriteLoopPIDRecord; keep started_at only if another reader needs it. Replace LoopSpec.BootTime with BootID func() (string, bool), nil selecting proc.BootID. In unmarkedExitCause, when the recorded and current boot ids are both known and differ, return \", and the machine has booted since the loop started, so a stop of the machine likely ended it\"; when both are known and equal, return \", and the machine has not booted since the loop started, so a kill of the loop and its keeper that no process can catch likely ended it, as by SIGKILL or the out-of-memory killer\"; otherwise keep the old two-cause clause. Pass the record's BootID from deadLoopStop and an empty id from loopExitedEarlyStop and the two loop_unix_integration_test.go call sites. Update the runLoop doc, the LoopPIDRecord field doc and internal/shedverbs/doc.go's unmarked-exit paragraph to match. In loopwait_test.go TestUntilStop_DeadLoopStopsLoopExited replace the startedAt and bootTime fields with recorded and current boot ids and turn the three UnmarkedExit rows into a different-id row, a same-id row and a recorded-id-with-unknown-current row with the new wording; in loop_test.go TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog assert the pid record's BootID equals a stubbed LoopSpec.BootID value instead of StartedAt, and drop the first-read pass-through clock in the two panic tests if the loop no longer reads Now at start. Replace TestBootTime in internal/proc/proc_linux_test.go with a test that BootID reports true and a non-empty id equal across two calls."
  - id: F2
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/shedverbs/loopwait.go#loopExitEvidence
    summary: A loop's reconciliation mark counts as exit evidence, so a loop that reconciled orphaned steps and then died unmarked gets that mark as its loop exit and never the unmarked-exit diagnosis.
    fix: "In loopExitEvidence, split the loop's marks into informational ones, those whose text after the loop prefix opens with \"reconciled the steps whose process is gone: \", and the rest. Take the `len(marks) > 0` branch only when a mark outside the informational set exists. When only informational marks exist and the loop started, write \"; loop exit: not recorded, since no process marked the loop's exit\" plus unmarkedExitCause's clause, then \"; loop marks: \" and the informational marks joined by \" | \". Update the loopExitEvidence doc comment and the marks paragraph of internal/shedverbs/doc.go to say a reconciliation mark is no exit record. Add a row to TestUntilStop_DeadLoopStopsLoopExited in internal/shedverbs/loopwait_test.go whose loop log holds the starting mark of dead-1 and then \"lym shed: loop dead-1 reconciled the steps whose process is gone: stopped shuttle runs: a:1:plan\", and whose wantCause carries the not-recorded clause followed by that mark."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loop_test.go#TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog
    summary: The test discards the loop's exit code, so the documented return of 1 from endWithoutEnvelope is unpinned, as the mutation survivor at loop.go:545 shows.
    fix: "In TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog add a wantCode int field, 1 for the refusal row and the code the loop returns after a failed envelope write for the envelope row; capture the code clihelp.Execute returns and report t.Errorf(\"step exit code = %d; want %d\", got, tt.wantCode) when it differs."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedverbs/loopwait_test.go#TestUntilStop_LoserWaitsOnWinner
    summary: The lock-loser branch of runLoop, the realistic early exit, marks "could not take the loop lock, so another loop owns the run" in the loop log, and no test asserts that mark.
    fix: "In TestUntilStop_LoserWaitsOnWinner, after the existing assertions, read the loop log and assert it holds a line opening with loopLogMarkPrefix, the losing loop's id and \" ends without an envelope: could not take the loop lock, so another loop owns the run\"."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedverbs/doc.go:155
    summary: The package doc's list of what the pid file holds omits the field the diff adds, and its "start time" means the process start-time token, so a reader cannot tell the two starts apart.
    fix: "In internal/shedverbs/doc.go change the line \"The pid file holds the loop id and the loop's own pid and start time.\" to name the new field, for example \"The pid file holds the loop id, the loop's own pid and process start time, and the boot the loop started in.\" after F1's fix, or \"... and the wall-clock time the loop started.\" if StartedAt stays."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedverbs/doc.go:225
    summary: The package doc names only a refusal or a lock another loop holds as the source of an early end, though a directory the loop cannot create and a pid file it cannot write mark one too.
    fix: "In internal/shedverbs/doc.go change \"Such an end comes from a refusal or a lock another loop holds.\" to \"Such an end comes from a refusal, a lock another loop holds, or a directory or pid file the loop cannot write.\""
---

### [MEDIUM] The unmarked-exit cause rests on a wall-clock boot time and is stated as fact

**Location:** internal/shedverbs/loopwait.go#unmarkedExitCause

**Issue:** `unmarkedExitCause` compares `proc.BootTime` with the pid record's `StartedAt` and prints a definite cause, "so a stop of the machine ended it" or "so a kill … ended it".
On Linux, `/proc/stat`'s `btime` is the current real time less the time since boot, so a clock step moves it.
NTP or chrony `makestep`, or a WSL2 or VM clock resync after the host sleeps, can step the clock forward after the loop starts.
Once the step exceeds the time between boot and the loop's start, `boot.After(startedAt)` holds and the note names a machine stop that never happened.
On Windows, `time.Now().Add(-windows.DurationSinceBoot())` shifts with a clock step the same way.
`GetTickCount64` also survives a Fast Startup shutdown, so a real machine stop reads as a kill "as by SIGKILL or the out-of-memory killer", a Linux-only wording.
A kill followed by a reboot also cannot be told from a reboot that killed the loop, so even a correct comparison only supports "likely".
The issue asks for a halt note that can name the cause, and a confidently wrong cause defeats that ask.
Linux's `boot_id` changes only at a boot, so it removes the clock dependence.
The hedged wording covers the residual doubt, and Windows keeps the old two-cause clause.
The handler, the code-correctness lens and the assistant each raised this finding independently.

### [MEDIUM] A reconciliation mark hides the unmarked-exit diagnosis

**Location:** internal/shedverbs/loopwait.go#loopExitEvidence

**Issue:** When `LoopSpec.ReconcileOrphanedSteps` returns a note, `runLoop` marks "reconciled the steps whose process is gone: …" under its own loop id (`loop.go`, after `OnStart`).
`loopLogLinesOf` counts every line with the loop's prefix as a mark, so `loopExitEvidence` takes its `len(marks) > 0` branch and prints that note as the "loop exit".
The note is no exit record.
A loop that reconciles runs after a dead loop, which is where an uncatchable kill or a machine stop is most likely to recur.
If that loop and its keeper then die unmarked, the halt note never reaches `unmarkedExitCause`, against the board entry's ask that every path ending the loop without an envelope leave a nameable cause.

### [LOW] The exit code of a loop that ends without an envelope is unasserted

**Location:** internal/shedverbs/loop_test.go#TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog

**Issue:** The doc comments of `endWithoutEnvelope` and `runLoop` both state that the loop returns 1.
The new test discards the result of `clihelp.Execute`.
The mutation report's survivor at `loop.go:545`, the constant in `endWithoutEnvelope`, shows that no test kills a changed exit code.

### [LOW] The lock-loser's mark is unasserted

**Location:** internal/shedverbs/loopwait_test.go#TestUntilStop_LoserWaitsOnWinner

**Issue:** Losing the loop lock is the early exit a second waiter hits in practice.
`TestUntilStop_LoserWaitsOnWinner` reaches the branch, per the reach report, but asserts only the winner's envelope and the started children.
The refusal row pins the shared helper, so only this branch's reason text is unpinned.

### [LOW] The package doc's pid-file shape omits the new field

**Location:** internal/shedverbs/doc.go:155

**Issue:** The line "The pid file holds the loop id and the loop's own pid and start time." predates the new field.
Its "start time" is the process start-time token of `proc.TreeRecord`, not the wall-clock start the diff adds.

### [NIT] The package doc's list of early-end sources is incomplete

**Location:** internal/shedverbs/doc.go:225

**Issue:** `endWithoutEnvelope` is also called for a directory the loop cannot create and a pid file it cannot write (`loop.go`, the `MkdirAll` and `WriteLoopPIDRecord` branches).
The doc names only a refusal and a lock.

## Rejected

- Assistant F1, as BLOCKING: the directory-creation and pid-write branches share `endWithoutEnvelope`, which the refusal row pins, and they differ only in literal reason text. The lock branch and the exit code are kept as F4 and F3 at LOW. PATTERN-test-speed sets no per-branch coverage rule.
- Assistant F2 and code-tests F2, coverage of `proc.BootTime`'s failure paths and of the Windows implementation: the Linux failure paths sit behind the fixed `/proc/stat` path, and adding a seam only for tests is unwarranted. A Windows test could only restate the one-line expression. F1's fix replaces the function either way.
- Code-scope lens: no findings, and no finding rejected.

## Focus conclusions

- `runLoop` early returns: the directory, lock, refusal and pid-write returns now mark the loop log. The envelope-write failure marks too, a signal keeps its existing mark, and an unrecovered panic in `OnStart` or `OnExit` reaches the keeper's exit mark and the crash excerpt. No path ends silently. The one gap is F2, where an informational mark masks the diagnosis.
- `unmarkedExitCause` and the whole-second `btime`: the truncation is no defect, since a loop never starts within a second of a boot. The Windows sleep accounting and a wall-clock step are F1.
- `StartedAt` with `omitzero`: older files decode as unknown and fall back to the old wording. `stepUntilStop` rewrites `*pid` and keeps the field. Pair teardown restores the whole record it read. The `landingdeps` reader ignores the field. No defect.
- `loopAgreesWithState`: held beside busy with cause `background` is the expected pair of a turn end whose unawaited shells still run. Held beside busy with cause `turn` still logs, so a stuck turn stays visible. Removing the false pair makes the issue's repeated warning resolve itself, which meets the third ask.
- `proc.BootTime` on Linux: an unreadable file, a missing `btime` line and a parse error each report false. The doc contracts on the two platforms agree. The defect lies in how the result is used, F1.
- Scope: every change traces to one of the three "What to do" points or is forced by one, and every point has a change that builds it.
