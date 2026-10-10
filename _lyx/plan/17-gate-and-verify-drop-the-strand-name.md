# Card 17 — gate-and-verify-drop-the-strand-name

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli/test_integration_test.go`
- `internal/gatecli/doc.go`
- `internal/verifytree#Verify`
- `internal/verifytree/verify_integration_test.go`
- `internal/verifytree/doc.go`

**Uses:**
- `internal/agentname#StrandNameEnv`

**Intent:** Keep an implementer's or Master's gate or verify command from handing the strand name to the tests it runs, so a test that drives `lyx fabric`, in-process or through a `lyx` subprocess, never meets the new role guard (D7).
`runTest` builds the child's environment from `os.Environ()` minus the strand-name variable before the slot's own entries are added, in the slotted and the unslotted branch alike; `Verify` does the same for the environment it hands `verifyrun.Run`, so an unslotted run no longer passes nil and inherits.
`doc.go` of each package states the dropped variable beside the slot environment.
Bound: this opens nothing the guard's bound does not already concede, since a test is code the fork writes, which could clear the variable itself.
`TestGateTest_Scenario` gains a row running the gate with the strand name set and a stand-in that records its environment, asserting the variable is absent; `TestVerify_Scenario` gains a row whose command prints the variable into a file and asserts it is empty, with the variable set by `t.Setenv` on the subtest's own goroutine, so no existing fabric test changes.

**ImpactSummary:** Two functions each drop one environment entry from the processes they spawn; no signature or caller changes.
