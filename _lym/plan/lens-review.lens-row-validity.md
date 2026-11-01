```yaml
lens: row-validity
outcome: reported
```

Task: review the plan's test rows through the row-validity lens.

**Findings**

1. **MEDIUM** · Card 10 (`TestGateRunner`, tier1), first row · The row can't do what it claims, and the fix it needs comes from a later card.
   - The row wants `phase.json` to read `running tier0` "while a held stand-in ran", and it also wants an exit file `{7, false}` from the same stand-in.
   - A held stand-in sleeps until it is killed (`internal/gatecli#runStandIn`, `if hold { select {} }`). It never exits 7, so the exit file can never read `{7, false}`.
   - The variable that would let a stand-in run for a while and then exit, `helperDelayEnv`, is created by card 11. Card 10 runs before it, so the row breaks plan order.
   - Fix: split the row into a held run that checks the phase and a plain run that exits 7, or move `helperDelayEnv` and its `runStandIn` change into card 10.

2. **MEDIUM** · Card 11, D2 "a call with no strand waits for slots without a cap" · The planned row would still pass if the code were wrong.
   - The rewritten "slots all" row checks that a call with no strand and one slot held "still waits after several caps".
   - Card 11 also removes `helperSlotWaitEnv`, so the only cap the wrong code could fall back to is `DefaultAwaitCap` (250 s). A call capped at 250 s would also still be waiting after a few hundred milliseconds.
   - Fix: give that row a short `helperCapEnv` (for example 50ms) and no strand. A front that wrongly caps a no-strand call at the resolved cap would then answer 75 and fail the row.

3. **MEDIUM** · Card 5, the Constraint that `runDeploy` and `runRowCommand` drop the strand · No row fails if the wiring is missing.
   - `TestCommandEnv` tests only the helper. If `runDeploy` or `runRowCommand` kept passing a nil environment, no planned or existing test would fail. Evidence: `internal/battencli#TestRunDeploy_WritesTheLogAndReportsExitCodeAndTimeout` runs parallel with no strand set.
   - `TestCommandEnv` is also a test of one symbol where the behaviour belongs at the caller.
   - Fix: add a row to the deploy test, and one for `runRowCommand`, that runs `test -z "$LYM_STRAND_NAME"` with the strand set. That needs a non-parallel subtest or an injected environment.

4. **LOW** · Card 11, the `helperLaunchEnv=none` row · The "refuses as claimed" assertion can't be told apart from the lease refusal, so D1's late-claim criterion goes unreached.
   - After a start failure, the front has already expired the lease. `runDetached` checks "no lease, passed lease, existing `process.json`", so a later `run-detached` call can refuse on the passed lease first.
   - Either the row's "as claimed" text fails, or it never reaches the exclusive-create branch. That branch is D1's "a runner that claims after the front's own exclusive create refuses".
   - Fix: state which refusal the row expects. Reach the claim collision under a live lease, as `TestGateRunner_Refusals` does with state it builds by hand.

5. **LOW** · Card 11, the `helperLaunchEnv=late` row · The timings race.
   - `helperLaunch` "late" starts the runner two seconds late, and the row uses "a 2s claim bound past the delay". With a bound equal to the delay, the claim lands right at the bound, so the row can flake into the start-failure branch.
   - Fix: a claim bound clearly above the delay, such as 4s for a 2s delay inside a 6s cap.

6. **LOW** · Card 11, the "key directory's lock held by the test" row · The card never names the lock file.
   - The row has to take the key directory's lock, but the card gives `detachKeyDir` and not the lock file's name or how it is locked (`internal/lock`).
   - Fix: name the lock file, and the helper the test takes it with, on the card.

7. **LOW** · Card 13, D6 "a lease that passes while a `--wait` gate line runs … frees its slots" · The "frees its slots" half is untested.
   - The `lym` stub on `PATH` takes no gate slots, so the row checks only that the stub is gone.
   - Fix: hold a slot from the stub (or assert on the pool) and check it is free afterwards.

8. **LOW** · Cards 10 and 11, D1 "on Windows, a front joins a live runner and supersedes one whose pid was reused" · Only the reading itself is tested.
   - `TestStartReading` covers how the start reading is formed. No row covers the join or supersede decision on Windows, and every `TestGateTest_DetachedRun` row is Unix-only in practice.
   - Fix: state the gap as a bound, or add a tier0 row that drives the join predicate with injected readers.

9. **LOW** · Card 4, D4 "an explicit `--cap` wins" · The only proof is that the test would be slow.
   - `TestAwaitVerb`'s `--cap 50ms` row runs outside any worktree, where the default is also `DefaultAwaitCap`. Code that ignored `--cap` would only make the row wait 250 s; it would not fail an assertion.
   - Fix: run one row inside a git worktree with a resolver answer that differs, or assert the call ends within a bound.

Tool calls made:
- One Bash call read `internal/agentname`'s `ValidateShortname` (to check that the `sh:...` strands in card 3 parse), searched `internal/hubforge` for `AddPair` (card 11's pair row) and read `internal/verifyrun/testmain_test.go` (card 6's helper role). My context held none of these.
