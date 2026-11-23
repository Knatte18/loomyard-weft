```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (the `run.go` fallback to a fresh start, plus the new and extended tests) through the fix-code-correctness lens.

no findings

Traces I ran, with the code in my context:

- **`start`, resumed attempt dies with its strand removed, `StartRetries` 1.**
  - Before the loop, `spec` is the resumed spec and `freshSpec` is the original spec plus the carry-over block. `freshWarning` and `freshCopied` hold the pre-resume values.
  - Attempt 1 returns `run != nil` and `startRetryable` holds, so the branch sets aside `run.runDir` and `pred.runDir`. It then switches `spec`, the warning, the copied flag and `resuming` back to the fresh values.
  - Attempt 2 launches the fresh spec: no `ResumeSessionID`, the prompt as given, and `SkipSkillLoadTurn` false.
  - On success, `takesPlace` sets the predecessor aside again; `setAside` skips a record already marked `NoResume`.
  - `resuming` is now false, so the notifier runs when `carried` is non-empty.
  - A non-retryable failure or the last attempt still returns the error unchanged.
- **`start`, nothing to resume.** `spec = freshSpec` reproduces the old `spec.Prompt += carryOverBlock(carried)` result.
- **`start`, a resume that succeeds.** It uses `resume.spec` with no carry-over block and no notice, and `launch` still records `carried`.
- **`TestStart_FailedResumeRetriesAsAFreshSession`.** With the defect in place (retry keeps the resumed spec), `PrepareCalls[1].Spec.ResumeSessionID` would be `interruptedSession`, so the test fails. The `NoResume` loop covers both the failed attempt and the earlier run, and the successful run stays false.
- **The carried-instructions row.**
  - A prompt with the block appended fails the existing `wantPrompt` equality.
  - A notice sent on a resumed start fails `len(notices) != 0`.
  - A record that dropped the carried instructions fails the `slices.Equal` check.
- **`TestRunner_RetireInterruptedRuns_SetsAsideOnlyResumableRuns`.** `os.ReadDir` sorts the entries, and `unwritable` sorts before `waiterless`. A scan that aborted on the first failed write would leave `waiterless` unretired, so the count would be 2 instead of 3. That makes the test deterministic against the defect.
- **`TestAttach_RecordsASupersededRunningCandidateAsInterrupted`.** Negating `c.state.Outcome == runOutcomeRunning` makes the running candidate's `Interrupted` false and the died candidate's true. Both row assertions fail.
- **`TestStop_RecordsTheStopBeforeRemovingTheStrand`.** Both `Run.Stop` and `StopStrand` leave `Interrupted` false, so the new assertion holds for both forms. It would fail if `StopStrand` passed `true`.
- **Callers.** No signature changed; only `start`'s local flow changed. `Start`, `StartGated` and `StartSession` are unaffected.

One thing outside the fix's scope: a resumed attempt that fails before a run exists (`run == nil`, such as a `Prepare` error) is neither set aside nor retried fresh. This was already so before the fix, and the review did not ask for it.

Further tool calls: none made, because my context held every changed file and the review.
