```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 proposal, the two lost-writer subtests added to `TestEngine_Resume_Writers`, and report where it is wrong or breaks something.

**Findings**

1. **LOW**, `resume_test.go`, both new subtests: the "no notice sent to the chair" check can pass when the defect is present.
   - The check is `len(chair.sent) == 0`, read after `Resume` returns. The sender delivers asynchronously (`sender.start(live.Chair)`), and `join` calls `sender.stop()` first, so a queued but undelivered notice would still leave `chair.sent` empty.
   - In the missing-outputs subtest, the realistic defect (counting a writer with missing outputs as finished) queues a notice and never fires the ender. `Resume` then blocks on the unreleased chair, so the test fails by hang or timeout instead of by assertion. It does fail, but not cleanly.
   - In the Accept-fails subtest, a defect that queues a notice and also ends the step would slip past the `chair.sent` check. `EndedStep` and `AcceptErr` would still be asserted, so the test would pass.
   - The proposal is not wrong. It is weaker than its wording ("without the chair being sent a notice") implies.

2. **LOW**, Accept-fails subtest: it never asserts that `Accept` ran, unlike the missing-outputs subtest, which counts calls. `AcceptErr` being set already proves a call happened, so this is not a coverage hole.

**No finding**

- The proposed assertions match what `Resume` does for the missing-outputs case. `allExist` is false, so `Accept` is short-circuited and never reached. The step ender fires, and a `seatRun` with no handle leaves `StrandGUID` and `Outcome` empty.
- `join` takes the `ender.signal` branch deterministically. It stops the chair once (`stopCount() == 1`), and the stop releases the chair as died with a nil error, so `Resume` returns a nil error. The gate run passed (126 passed, 0 failed).

**Tool calls:** none made. The context already held everything this lens needed.
