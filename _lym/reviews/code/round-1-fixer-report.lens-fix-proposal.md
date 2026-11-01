```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 and F2 proposals the fixer carried out as proposed. No findings.

- **F1, dirty-checkout row.** I traced it step by step.
  - Wake 2 raises the flag, so `judgeDone` stops `guid-1` and `refuseDirtyDone` clears the flag.
  - `ensureSession` finds `guid-1` not live and starts with resume id `session-1`, giving `strand-2`.
  - The mail goes to `strand-2`.
  - Wake 3 reads `DoneStamp` 0 with `guid-2` live, and the third sleep cancels the context.
  - That gives resumes `["", "session-1"]`, stops `["guid-1"]`, one mail and `DoneStamp` 0, which matches the row.
  - A wake-specific break, such as dropping the mail or ending the call on the dirty branch, would fail the row's `wantMails`, `wantErr` or `check`.
  - `DirtyPaths` is first called on the wake path here, so the shared `dirty` fixture does not interfere.
- **F1, damaged-record row.** `judgeDone` stops `guid-1`, sees no dirty paths, and `CheckRecord` returns findings, so the result is Stuck with Halt. `endSession` finds `guid-1` not live and does not stop it again, so stops stay `["guid-1"]`. The reason holds both the finding and `decisionRecordRestoreWayForward`, so the restore-steps assertion holds. The test harness checks Halt on every Stuck outcome.
- **F2.** I tried to build a case where the fall-through is wrong and found none.
  - The sampler session itself with an unreadable record: `parent.Name` differs from `strandName`, so it is refused with the sampler-read message.
  - A caller with an empty parent name: `strandName` is non-empty and differs from the empty name, so it is refused the same way.
  - Parent resolution failing while the sampler read also failed: this refuses with the sampler-read message.
  - The `sampler &amp;&amp; samplerErr == nil` guard stops a half-valid pair from admitting a caller as the sampler.
  - The existing "sampler session check failure" row still refuses, because its default parent name is empty.
  - The new row ("the recorded parent with an unreadable sampler record") reaches a zero exit only when the early refusal is gone, so it fails with the defect in place.

Tool calls: none (0 of 6 used); the context already held every changed file.
