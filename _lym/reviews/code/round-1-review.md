---
verdict: APPROVED
---

No finding survives consolidation.
None of the four lenses (code-correctness, code-affected-set, code-tests, code-scope) reported a finding, and the handler's holistic review found none.

## Review range

`git diff 5f9e1e95d93e6ea942d4c5a72f64401113b6101e..HEAD` holds one commit, f84f1a937.
That commit changes only `internal/creelengine/machinemail_test.go`.

## Focus entries, each driven

1. **The race after the reorder.** Closed.
   - Every post and the hold report run before `Scheduler.Start`.
   - With `machineHook` still nil, `Store.do` reads `machineWake` and clears it without calling a hook (`pool.go:191-192`, `pool.go:212`), so no wake is left over.
   - `Start`'s own wakes go to the delivery, chase and outbound workers.
   - The scheduler's `machineWake` channel is new and empty (`deliver.go:73`).
   - Only a post or a reply to another machine sets the store flag (`crossmachine.go:123`, `crossmachine.go:192`), and the subtest makes none after the start.
   - The return notices go to the local orch, so they set no machine wake.
   - The first pass reads the store and the clock before it calls `After` (`machinemail.go:304`).
   - The test's `AwaitWaitsWhere(1, isHourWait)` returns only once that `After` has recorded the wait (`creelfake.go:397`), so no pass straddles `Advance(time.Hour)`.
   - `Advance` fires the timer due exactly at the hour (`creelfake.go:417`).
   - The second pass returns both messages and arms no timer.
2. **The subtest still checks what its name states.** Yes.
   - The scheduler is started.
   - No take runs.
   - The waiter is answered `OutcomeReturned` at the hour.
   - The held message returns with `peer-unapproved` and the bare message with `bridge-unsettled`.
   - One notice per message is checked.
   - Every assertion is unchanged.
3. **Claims in `_lym/landing/summary.md` and the new comment.** All are borne out by the code.
   - "With no hook installed yet, the posts wake nothing": true, per entry 1.
   - What the first pass arms: true, per entry 1.
   - A post after the start still wakes the loop:
     - `APostToAnotherMachineStartsNoHubWorkerAtTheSchedulersStartOrOnThePost` covers the loop arming an hour wait on that wake.
     - `TwoMessagesDueTogetherAreReturnedInOnePassAndTheLoopNeverWaitsUnderASecond` covers those messages being returned at their hour.
4. **The Root-Cause trailer and the commit message.** They match the defect the diff removes.
   - On the merge base, the second post wakes a pass that can read the store before the advance and read the clock after it.
   - That pass then waits on `max(0, machineHourFloor)`, a one-second timer that nothing advances past.
   - The first pass's timer had already been dropped when the second post's wake arrived.
   - The reorder removes the second wake.

## Holistic checks

- **Scope.** The diff makes the change the parent's `Added after Discussion` entry asks for: a test-only reorder with no production change.
- **Merge danger.** The `two-way` call fits a test-only diff, and the blast radius `creelengine` covers what it can break.
- **PATTERN rules.**
  - `PATTERN-test-speed`: the test stays at tier 0 with `t.Parallel`.
  - `PATTERN-test-stand-ins`: the fake stands in only for the clock.
- **Tooling names.** The summary names no tooling.
- **Substrate.** `lym gate test ./internal/creelengine -- -run TestStore_MachineOutbound -count=30` passed 360 of 360 at tier 0.
