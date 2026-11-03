```yaml
lens: fix-proposal
outcome: reported
```

**Lens fix-proposal.** I challenged each finding the fixer fixed as proposed (F1–F6). There are two findings, both LOW.

**LOW — F4, `internal/hibernateengine/watch.go` `carry`, the added resuming reason**
- Summary: the reason says the wake reconcile "did not run", which may be false.
- Evidence: the sequence writes `PhaseResuming` (`advance`) before it calls `reconcileAtWake`. A dobby that dies or re-execs after the reconcile finished but before `finish(..., PhaseDone)` leaves a `resuming` request whose reconcile did run. The takeover then tells the operator that it did not.
- Case: the process is killed between `reconcileAtWake` and `finish`. The notice and the stored `Reason` assert a skipped reconcile, but the origin reconcile and the pair pauses are already done.
- Fix direction: word it as "may not have run" or "could not be confirmed". `lym cleanup --apply` is still the right pointer, because it is a safe re-run and `wake.go` already names it for reconcile failures.
- Severity: the rubric's MEDIUM ("states behaviour the code does not hold") comes close, but I rate it LOW because the pointer and the recovery are still correct.

**LOW — F1, `internal/fabriccli/hibernatedaemon_test.go`, `TestHibernateEndedPhases_MatchTheEnginesTerminalPhases`**
- Summary: the pin works from a hand-listed set of engine phases, so a new terminal phase is not caught.
- Evidence: the test enumerates the eight `Phase*` constants itself. A ninth terminal phase added to `Request.Terminal()` (and the constants) is not in that list. The first loop never checks it, and the second loop only checks entries already in `hibernateEndedPhases`.
- Case: the engine adds `PhaseAbandoned` and `Terminal()` is updated. `hibernateEndedPhases` stays stale and the test stays green. This is the scenario the review's F1 gave for wanting a test, which was to catch a "new terminal phase in the engine".
- Fix direction: the engine could expose its phase list (an `AllPhases` or a `Terminal`-phase list). The test would then range over that instead of its own copy.

**Proposals I challenged and found sound:**
- **F5, the touch after `running = false`:**
  - The rewrite happens under the request lock and keeps the content, so it cannot clobber a concurrent write.
  - It cannot loop. A terminal request takes nothing up, and every non-terminal exit of `carry` I traced either leaves the file not ours or comes from a write failure, in which case the touch fails too and fires no event.
- **F3, `MkdirAll` in `NewDobbyCarrier`:**
  - It is correct. Only the prime's dobby calls it, and it adds a failure path (a read-only Bolt scratch directory) that the verbs already share.
  - The new assertion only helps if the forged test hub lacks the hibernate directory beforehand. The non-test code creates it only in `hibernatecli`: `HibernateDirName` is used in `deps.go` and `anchordeps.go` only, and neither creates it. So the assertion fails without the fix. I did not run it with the fix removed.
- **F2 and F6:** no case against them.

**Calls made:**
1. `grep HibernateDirName` over `internal`, plus `hubforge` and `doc.go` greps: to check that nothing else creates the hibernate directory, so the F3 test pins the fix. The fixer's context lacked this.
2. `grep` in `hibernateengine/wake.go` and `cleanupcli`: to check that `lym cleanup --apply` is the real recovery for a skipped reconcile (F4). It is, and `wake.go` already points to it for reconcile failures.
