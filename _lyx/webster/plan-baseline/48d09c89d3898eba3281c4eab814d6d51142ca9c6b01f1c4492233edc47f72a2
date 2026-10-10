# Card 4 — verifytree-slot

**Edit:**
- `internal/verifytree#Verify` -> `func Verify(ctx context.Context, p Paths, site Site, command string, timeout time.Duration, slots *gateslot.Pool) (Result, error)`
- `internal/verifytree#Marker`
- `internal/websterengine#NewVerifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/loomshed#NewVerifyGate`
- `internal/webstercli#websterCLI.runVerify`
- `internal/verifytree/verify_integration_test.go`
- `internal/verifytree/marker_test.go`
- `internal/loomshed/verifygate_integration_test.go`
- `internal/verifytree/doc.go`
- `pattern/PATTERN-verified-tree.md`

**Uses:**
- `internal/gateslot#Pool.Acquire`
- `internal/gateslot#Lease.Env`
- `internal/gateslot#Lease.Release`
- `internal/verifyrun#Run`

**Intent:**
Make `verifytree.Verify`, the path every plan-verify site already takes, acquire a gate slot before it spawns (D4), so no site gains a second runner (`PATTERN-verified-tree`).
After the dirty and skip checks pass, and only then, `Verify` writes the running marker in a waiting state with its wait start, acquires a slot from `slots` under the caller's `ctx` with a `Holder` naming `p.Worktree` and `site.Label`, rewrites the marker as running with a fresh start time, and only then starts the `timeout` context, so the verify timeout counts from the moment the slot is held (D6).
The shell gets `lease.Env(os.Environ())`: `-p=<cap>` appended to any existing `GOFLAGS`, plus the inheritance variable naming the held slot, so a `lyx gate test` inside the verify command runs in the parent's slot without deadlocking.
The lease is released when the run ends, whatever its outcome; a cancelled wait returns the cancellation as an error and writes no record, as a cancelled run does today.
A nil `slots` runs unslotted with the parent environment, the form a standalone run with no hub and a unit test use.
`Marker` gains a `State` (`waiting` or `running`; a marker written before this field reads as running) and a `WaitStarted` time; `ReadMarker` is unchanged otherwise.
Every caller passes nil here; the following cards wire each site's pool.
`verify_integration_test.go` gains rows: with a one-slot pool held by the test, `Verify` leaves a waiting marker, runs once the slot is released, records the pass, and its command sees `-p=<cap>` in `GOFLAGS` appended to an existing value and the inheritance variable; a timeout shorter than the wait does not fire while waiting.
`marker_test.go` gains the state round-trip and the legacy marker reading as running.
`doc.go` and `pattern/PATTERN-verified-tree.md` state the slot step, the waiting marker and when the timeout starts.

**ImpactSummary:** Every `verifytree.Verify` caller (webster gate, round gate, Publish and Finalize, `lyx webster verify`) gains a nil argument and keeps today's unslotted behaviour until its own wiring card.
