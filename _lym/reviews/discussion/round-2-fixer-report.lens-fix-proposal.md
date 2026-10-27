```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer proposals the fixer carried out, through the fix-proposal lens.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F9, F10 (the fixer took the proposal's second option), F11, F12 and F13. F8 was fixed differently and is out of this lens.

**[MEDIUM] F10 + F3, Decision 5: moving `dobby.out` rotation into the lock holder makes rotation impossible on Windows.**
- Changed text: "The process that takes the lock, at each start, fresh or re-exec, rotates `dobby.out` …" and the bound "A rotation whose rename fails, as on Windows when another process holds the file open, logs its error … until a later start rotates it."
- Case: on Windows, `SpawnDetached` gives the dobby `dobby.out` as its own fd 1 and 2. Windows has no exec, so the F3 reopen does not run there and the holder keeps that handle. The holder's own rename of the file it holds then fails at every start, not only when another process holds it.
- So no later start ever rotates it, and `dobby.out` has no bound on Windows. The original spawn-time rotation, and F10's first option (rotate at spawn when `Live` reads the lock free), rotated while no process held the file.
- Evidence: Go opens files with `sharemode := uint32(FILE_SHARE_READ | FILE_SHARE_WRITE)` (`$GOROOT/src/syscall/syscall_windows.go:395`). Without `FILE_SHARE_DELETE`, a rename of an open file fails. `internal/selfexec/identity_windows.go:3` confirms Windows never execs.
- The bound sentence's "until a later start rotates it" is false on Windows.

**[LOW] F13, Decision 6 rationale: "eleven minutes, the interval plus the loop's one-minute idle ceiling" holds only while `notice_probe_s` ≤ 60.**
- The loop's wait is `max(wait, d.probe, wakeFloor)`, and `nextWait` caps only the backoff at `idleCeiling` (`internal/dobbyengine/loop.go:374-378` and the loop body). So the idle tick gap is `max(1 min, notice_probe_s)`.
- The recipe value is 30 s (`contracts/recipes/batten-recipe.yaml:76`), so the claim holds today. A hub that tunes `notice_probe_s` above 60 breaks it.
- The rationale should name the gap as the interval plus the larger of the idle ceiling and the probe interval.

**[LOW] F2, Decision 2: making `recordStart` share the re-exec test extends the accepted "ticks-less record reads as a re-exec" case from the lock wait to the full pass.**
- Case: a pre-change record holding pid P and no ticks, and a reboot whose fresh dobby gets pid P.
- That dobby now reads as a re-exec in `recordStart` as well, so it owes no full pass after the reboot. Before F2 it only got the long lock wait.
- It also writes its ticks into the record, so the misreading is permanent for that record.
- The record's bound covers only the wait ("so the first re-exec onto the binary that ships this change takes the long wait"). It should state that such a dobby also skips its post-reboot full pass, or that the ticks-less re-exec reading holds only for the wait.

No breakage found in F1, F3 (apart from the Windows interplay above), F4, F5, F6, F7, F9, F11 or F12.

Tool calls:
- Read `nextWait`, the `notice_probe_s` wiring and Go's Windows `syscall.Open` share mode: the context lacked the loop's wait cap and the Windows open semantics.
- Read the batten recipe's `notice_probe_s` value: the context lacked the configured probe interval.
