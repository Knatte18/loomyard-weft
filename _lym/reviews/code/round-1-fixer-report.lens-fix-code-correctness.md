```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1 test rows and the F2 `decisionVerb` change.

No findings.

**F2, `decisionVerb` (`decision.go`)**
- A parent caller with `samplerErr` set and `parent()` resolving to its own name skips both refusals. It falls through to the status and recording checks, so the append succeeds.
- A non-parent caller with `samplerErr` set refuses with "read the run's sampler session: …; way forward". The existing "sampler session check failure" row still gets that message, because its parent name is empty and the caller name is not.
- A caller that is `sampler` with `samplerErr == nil` takes the unchanged sampler branch.
- When `samplerStrand` returns `(true, err)`, the code now treats the call as a failed sampler read and does not trust the `true`. That is the safe reading.
- A failing `parent()` together with `samplerErr` refuses with the sampler-read message. A failing `parent()` alone keeps the old message.
- With the defect in place, the new "recorded parent with an unreadable sampler record" row returns a nonzero code and the table loop's `code != 0` check fails it.

**F1, the new `samplersession_test.go` rows**
- **Dirty-checkout row.**
  - Wake 1 is quiet.
  - Wake 2 raises the flag.
  - `judgeDone` stops `guid-1`.
  - `DirtyPaths` returns `main.go`.
  - `refuseDirtyDone` clears the flag and calls `ensureSession`, which resumes `session-1` as `guid-2` / `strand-2`.
  - It mails `strand-2`.
  - Wake 3 sees the live `guid-2`.
  - The third sleep exhausts the budget, so the call ends cancelled.
  - The expected values match this trace: resumes `["", "session-1"]`, stops `["guid-1"]`, one mail to `strand-2`, `DoneStamp` 0.
  - A dirty branch that dropped the mail, skipped the clear or ended the call would break the row.
- **Damaged-record row.**
  - Wake 2 flags a clean checkout, because `dirty` is empty and `DirtyPaths` returns nil.
  - `CheckRecord` returns the finding, so `judgeDone` returns Stuck with Halt.
  - `endSession` does not stop `guid-1` a second time, because it is no longer live.
  - Stops are `["guid-1"]` and resumes are `[""]`, as the row expects.
  - The reason asserts reuse the strings the existing damaged-record row already asserts.
- No signature changed, so no caller is affected.

Further tool calls: none.
