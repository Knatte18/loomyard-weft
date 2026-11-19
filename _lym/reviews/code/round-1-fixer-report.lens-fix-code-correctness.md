```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Walkthrough of each changed function and test, with one concrete input each:

- **`proc.BootID` (Linux):** an unreadable file returns `"", false`. A file holding only whitespace trims to `""` and returns `"", false`. A normal id returns `id, true`. Windows returns `"", false`.
- **`runLoop`:** with `LoopSpec.BootID` nil it uses `proc.BootID`. An unknown id stores `""`, and `omitempty` drops the field, so older pid files and Windows fall back to the two-cause clause. Dropping the `now` variable leaves no stale use.
- **`loopExitEvidence`:**
  - A log with a starting mark and only a reconciliation mark leaves `exitRecorded` false. It takes the `default` branch and appends "loop marks".
  - A log with a signal mark plus a reconciliation mark takes the exit branch and joins all marks.
  - A log with no starting mark yields no marks, so it takes the `!started` branch.
  - The prefix ends in a space, so a loop id like `dead-10` does not match `dead-1`.
- **`unmarkedExitCause`:** a different id gives the "booted since" clause, and an equal id gives the "not booted" clause. The old clause covers an empty recorded id (older pid file, Windows) or an unreadable current id.
- **Callers:** `deadLoopStop` and `loopExitedEarlyStop` pass the new argument. The two integration call sites pass `""`.
- **Tests that would fail with the defect in place:**
  - F2's reconciliation-only row would print "loop exit: lym shed…" without the fix.
  - F3's code assertion would fail if `endWithoutEnvelope` returned 0. The envelope row's `wantCode` of 0 matches `output.Ok`, and the gate passed.
  - F1's `rec.BootID` assertion would fail if `runLoop` did not record the id.
  - F4's new subtest would fail without the mark. It is deterministic because `LockGrace` is 1ms and the lock is held in-process. The gate passed.
- **`TestBootID`:** it covers the unreadable, empty and unstable-id cases.
- **Build:** `GOOS=windows go vet` on `shedverbs`, `pairteardown` and `proc` is clean.
- **Leftovers:** no `BootTime` or `started_at` reference to the loop pid record remains.

Calls made:
1. A grep for leftover `BootTime` or loop `started_at` references in Go, md and yaml, plus a `GOOS=windows go vet` of `shedverbs`, `pairteardown` and `proc`. My context held no repo-wide leftover search and no Windows build of `shedverbs`.
