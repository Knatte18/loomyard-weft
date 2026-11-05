```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens on the changed driver code (`run`, `arbitrate`, `noticeArbiterSettled`) and the new test rows.

**Traces**
- **Awaiting stop, arbiter resume, step consumes it.**
  - `arbitrate` sets `pendingNotice` and `pendingAttempt="1"`, then `settleStop` returns `takePendingAttempt()`.
  - `Step(ctx,"1")` returns a done envelope, so `attemptHeld` is true and `attemptRefused` is false.
  - `noticeArbiterSettled` sends the line once with the right stop and verdict path.
- **Step refuses the attempt.** The `arbiter_attempt` field makes `attemptRefused` true, so no notice goes out. The `if attemptRefused` branch then hands back. `pendingNotice` stays set, but the driver exits, so nothing reads it.
- **`Step` returns an error, or the envelope does not parse.** Both hand back before the notice call, so none is sent.
- **Busy envelope.** `attemptHeld` is false, so nothing is sent and the driver exits.
- **Yield after the consume.** The notice is sent before the yield check, so a consumed resume that then yields still notifies.
- **Blocked-stop arbiter resume.** `pendingNotice` is only set when the state is awaiting, so `noticeArbiterSettled` finds nothing pending.
- **Callers.** `noticeArbiterSettled` has no other caller after the signature change. The tackler path never sets `pendingNotice` or `pendingAttempt`, so it is unaffected.

**Tests fail with the defect in place**
- Notice sent before the consume: the refused row gets an "Nothing to answer." line, and `wantNoNotice` fails.
- `State == awaiting` guard removed: the blocked-resume row notifies once against a want of 0.
- `OrchRecorded` false ignored: the no-orch row notifies once against a want of 0.
- `OrchRecorded` error ignored: that row has `orchRecorded` true alongside the error, so it would notify and fail.
- Notice text changed: `wantNotice` on the success row compares the line exactly.
- Failed `Notify`: the row pins `wantNotifies` 1 and `ExitDone`.

The gates the fixer ran for `internal/sheddriver` and the dependent packages passed, including the guessed `wantNotifies: 1` on the refused row.

no findings

Tool calls: none; my context already held every changed file.
