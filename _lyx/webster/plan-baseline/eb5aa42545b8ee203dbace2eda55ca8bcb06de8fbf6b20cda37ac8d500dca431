# Card 17 — gate-and-verify-drop-the-strand-name

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli/test_integration_test.go`
- `internal/gatecli/doc.go`
- `internal/verifytree#Verify`
- `internal/verifytree/verify_integration_test.go`
- `internal/verifytree/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/agentname#StrandNameEnv`

**Intent:** Keep an implementer's or Master's gate or verify command from handing the strand name to the tests it runs, so a test that drives `lyx fabric`, in-process or through a `lyx` subprocess, never meets the role guard card 18 adds (D7).
This card lands before the guard, so no gate or verify run between the two cards hands a test the strand name the guard reads.
`runTest` builds the child's environment once, from `os.Environ()` minus the strand-name variable, before it branches on the slot, and every branch, the slotted, the inherited-slot and the unslotted one, adds its own entries to that one environment; `Verify` does the same for the environment it hands `verifyrun.Run`, so an unslotted run no longer passes nil and inherits.
`doc.go` of each package states the dropped variable beside the slot environment.
Bound: this opens nothing the guard's bound does not already concede, since a test is code the fork writes, which could clear the variable itself.
`TestGateTest_Scenario`'s subtest for each branch, `a free slot runs go test under the cap and exits with its code` for the slotted one, `an inherited held slot runs without acquiring` for the inherited one and `a target outside every hub runs unslotted and logs it` for the unslotted one, runs its gate with the strand name among the gate process's extra environment entries and asserts the stand-in recorded the variable absent; the gate process gets the variable through its own spawn environment, so the parallel test touches no process-global state.
`Verify` runs in the test's own process and `TestVerify_Scenario` is parallel, where `t.Setenv` panics, so a new top-level `TestVerify_DropsStrandName` in `verify_integration_test.go` sets the variable with `t.Setenv`, runs without `t.Parallel` under a comment naming that variable, and asserts that a verify command printing the variable into a file finds it empty, once per branch `Verify` builds its environment on: a row over a nil pool for the unslotted run whose nil environment the card replaces, and a row over a free one-slot `gateslot.Pool` in a temp dir, built as `TestVerify_SlotGate` builds its pool, for the slotted run whose `lease.Env(os.Environ())` would otherwise keep the variable; no existing fabric test changes.
The verifytree row of `test-budget.yaml` rises by one.

**ImpactSummary:** Two functions each drop one environment entry from the processes they spawn; no signature or caller changes.
