# Fixer-report, code round 1

All seven findings fixed as proposed.
Because F1, F3, F5 and F7 touch the same files, F1–F7 went into one commit (`22052e7a`) instead of one per finding; the self-check's doc fixes are a second commit.
`lym gate test --tier 1 ./internal/proc` passes, with the capped-run tests running and not skipped.

## Changes

- F1, F5: `scopeStartScript` in `internal/proc/memorycap_linux.go` is one constant expression over `selfCgroupFile` and copies the listing with the builtins `read` and `echo`, so it needs no `cat` from the caller's PATH.
- F2: the leftover test's poll deadline is a minute, in a shared helper `requireProcessGone`.
- F3: `runMemoryCapped` kills the scope after the command's own SIGKILL or when the scope's `memory.events` shows an `oom_kill` above 0 (`scopeHadOOMKill`, file named by `cgroupMemoryEventsFile`).
  `killed` stays true only for the command's own SIGKILL.
  The docs in `memorycap_linux.go` and `memorycap.go` say any kill at the cap.
- F4 with F3's case: `TestRunMemoryCapped_AKillOfAChildLeavesTheCommandRunningAndEndsItsScope` runs the hog as a plain child, asserts `killed` false, no error, output `<pid>\nsurvived\n`, and the sleep gone.
- F6: `killScope` opens `cgroup.kill` with `os.OpenFile(..., os.O_WRONLY, 0)` before it writes.
- F7: the rows of `TestRunMemoryCapped_EndsAChildOverTheCapByTheHostsMeans` assert `err` is exactly an `*exec.ExitError`.
- Self-check follow-ups: `doc.go`, the two file headers and the `RunMemoryCapped` doc line break now match the behavior.

## Self-check

- fix-code-correctness, LOW, `memorycap.go:11` line over 120 characters: fixed by splitting it at the clause.
- fix-consistency, LOW, same long line: duplicate of the above.
- fix-consistency, LOW, file headers of `memorycap_linux.go` and its test describe only the command's own kill: fixed.
- fix-affected-set, LOW, `doc.go:54` still says the kill follows the command's kill: fixed.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
