```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran each finding's own search and checked whether the fix covers every member.

**F1 (set: tests and sites under `helperOrphanEnv`).** The search found every site, and each is consistent.
- It found one stand-in gate at `test_integration_test.go:279-280` and two setters, at `:1566` (tier, rows 134 and 0) and `:1586` (build).
- The old literal `"1"` is gone everywhere. A non-`tier` or non-`build` value now disables the orphan, so no stale setter remains.
- The only other `TestGateTest_Scenario` row with a zero shell exit that could reach the `ErrWaitDelay` branch is the new `ZeroExit` row.
- No findings.

**F2 (set: callers of `runTiedChild`).** There are two non-test callers, `build.go:47` and `tierrun.go:228`.
- The new build subtest covers `build.go:47`.
- The tier subtest covers `tierrun.go:228`, on the failing and zero-exit paths.
- No findings.

**F3 (set: statements that the start-time check leaves a reused group id alone).** Rerunning the search found one more instance of the claim the review called false, outside the summary.
- LOW — `internal/proc/tied.go:85` says the kill happens "only while a live process still reports the recorded group id with a start time at or after the recorded one, so a reused group id is left alone."
  - This is the same overstatement the review called false against `tied_linux.go`: a process that started later and reuses the id passes the check.
  - The fixer corrected only `_lym/landing/summary.md`, so the summary now disagrees with this doc comment.
  - The review said "Leave the code as it is", and this file is outside the diff's reach. It is a sibling the fix left stale, so I report it as LOW and do not call it a defect of the fix.
- No other instance of the claim exists in the repository.

Unchecked findings: none.

Tool calls:
1. One Bash call ran three greps: `helperOrphan` / `GATECLI_TEST_ORPHAN` over the Go files, the "start-time check" / "reused group" / "reused pid" wording over the Markdown and Go files, and `runTiedChild(` call sites. My context held none of these results, since the review named only some sites.
