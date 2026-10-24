Task: challenge the reviewer's F1–F4 proposals as the fixer carried them out (lens fix-proposal).

1. **MEDIUM, `haltExitCode` in `internal/gatecli/run.go`: a cancelled-context halt after an earlier failure returns 1, not the first failure's code.**
   - F2's proposal (a halt with code 0 becomes 1) interacts badly with F3's rule that the first failure's code is kept.
   - Case: tier0 fails with exit 7, then tier6 passes (exit 0) while the context was cancelled without a caught signal. `haltExitCode(0, 7)` hits `haltCode == 0` first and returns 1.
   - The `case haltCode == SlotBusyExit && firstFailureCode != 0` branch shows the intent is to keep the first failure's code, which the rest of the call does.
   - Fix: the 0 case should return `firstFailureCode` when it is non-zero, and 1 otherwise.

2. **LOW, same function: a halt after the final run passed turns a complete, passing call red.**
   - `report` sets `halt` whenever `e.ctx.Err() != nil`, even for the last run of the plan.
   - A run that exits 0 after a non-signal cancellation, with nothing left to skip, now returns 1.
   - That is arguably correct, since the context was cancelled, but the call prints no message. `printFailedRuns` prints nothing when `failures` is empty, so the exit 1 is unexplained.

3. **LOW, `printFailedRuns` on the halt path: the closing line no longer says the call stopped early.**
   - The fixer dropped "every later run still ran" because it would be false after a halt.
   - Nothing replaces it, so a halt with `failures` non-empty prints `failed runs: tier0 (exit 7); the exit code is tier0's` and a reader cannot tell that later runs never ran.
   - Case: the refusal subtest, where tier3 never ran, prints the same line as a complete call.
   - The refusal's own JSON envelope sits in the same stdout, so this is only mildly misleading.

F1: no finding. The first failing summary line is also the exit code's tier, since a guard run comes after its listed tier in `planTierRuns`.

F4: no finding. The `^(TestGuard` prefix is the filter every guard run in that table gets, and no listed tier in these scenarios has one.

Further tool calls: none.
