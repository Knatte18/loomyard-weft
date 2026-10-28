```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each proposal the fixer carried out as written (F1–F4, F6–F12; F5 was the parent's call).

**Proposals I could not break:** F1, F3, F4, F6, F7, F8, F9, F10, F11, F12.
- **F1:** the allowlist check covers only the package's direct imports, so adding `creelengine` and `agentname` is enough.
- **F3:** this case runs before card 7's unanswered case. That keeps card 7's row "a `WaitReply` registered after the outcome is answered `OutcomeUnanswered`" true, because that row has no reply.
- **F4:** `releaseGoneHolder` never moves mail to a program; when the path ends at one it journals `undelivered` (`internal/creelengine/sweep.go:220`). So no path relied on `CREEL` being a known mailbox.
- **F12:** card 1's `TestAsker` rows cover the beam with its orch live and not live, the prime, and `m` with the beam's orch live. `TestParentOrchStrand` covers the pair walks, so the dropped rows are covered.

**[LOW] The backed-off chase clock drops some real running time, and no sentence says so**
- **Changed text:** card 7's clock paragraph: "a tick that reads it running, or a wake of the holder, returns the wait to `agedPersistInterval`".
- **Case:** a holder goes dormant, and its chaser backs off to the 8-minute ceiling. The holder then comes back live.
- **What happens:** the holder coming back live fires no wake. `Scheduler.wake` runs only through the store's wake hook (`internal/creelengine/deliver.go:61`), and the plan names no reed liveness event that signals it. So the return is seen only at the next backed-off probe, up to `chaseProbeCeiling` later. One more tick of `agedPersistInterval` is uncharged after that, because the charge rule needs two running ticks one interval apart.
- **Effect:** up to about 9 minutes of real running time per stopped spell is never charged, and a chase lands that much later than D10's "15 minutes of running time" suggests.
- **Why it isn't covered elsewhere:** the overview rationale's "a liveness change is seen within a minute while the clock runs" only covers a running clock stopping, not a stopped one restarting. The new test row asserts the drop back to one minute but not this lag.
- **Fix:** add one sentence to the overview's `chase-clock-cadence` and card 7's clock paragraph saying a stopped-to-running change is seen within `chaseProbeCeiling` and goes uncharged.

Calls made:
- Read card 1's `TestAsker` lines and grepped `TestParentOrchStrand` to check F12's claimed coverage; my context did not hold card 1.
- Grepped `creelengine`'s `lymcwd` imports, `scankit`'s allowlist check and the `Scheduler.wake` callers to check F1, and what a holder going live wakes for F2.
- Read the wake hook in `deliver.go` and the move path of `releaseGoneHolder` in `sweep.go` to check F4's case of mail going back to `CREEL`.
