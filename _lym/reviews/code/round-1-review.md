---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: "internal/fabriccli/anchordeps.go:43-44"
    summary: "hibernateEndedPhases repeats the engine's terminal phase values as string literals, which PATTERN-named-constants forbids"
    fix: "In internal/fabriccli/anchordeps.go replace the literal list done, failed, cancelled with one source of truth. Either move the Phase values, their Terminal test and the request file name into a stdlib-only leaf package that both internal/hibernateengine and internal/fabriccli import, or, when the import chain that anchordeps.go cites rules that out, keep the literals and add a test in a package that can import both which fails when fabriccli's list differs from the engine's terminal phases (an export_test.go seam in hibernateengine). Without either, record the departure with a Pattern-Departure trailer naming PATTERN-named-constants."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: "internal/fabriccli/anchordeps.go:240-256"
    summary: the new ended-phase rule of hibernateDaemonLive has no test, so an ended request could block fabric anchor --set again unseen
    fix: "Add a table-driven test of hibernateDaemonLive in internal/fabriccli (a package-internal test file beside anchordeps.go) that writes a request.json under the hub's hibernate directory and asserts the result for four rows, a waiting request naming a live carrier pid gives true, a pending request naming a dead pid gives false, a pending request with daemon_pid 0 gives false, and a done, failed and cancelled request that still names a live pid gives false. Use parentlock.ProcessAlive's own pid (os.Getpid with its start time) for the live carrier."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/beamcli/dobbywatch.go:129-131"
    summary: the hibernate directory may not exist when dobby opens its file watch, so a first request on an idle hub waits for the loop timer, up to a minute, not for the write
    fix: "Create the hibernate directory before the watch opens. In internal/hibernatecli/deps.go make NewDobbyCarrier call os.MkdirAll on hibernateDir(prime) (which already resolves through lymdirs.HibernateDirName) and return its error, since hubDobbyDeps builds the carrier before dobby's Watch closure runs. Add a row to the dobbyWatchDirs or hubDobbyDeps test that the prime's carrier wiring leaves the request's directory existing."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/hibernateengine/watch.go:67-70"
    summary: a request taken up in the resuming phase with ResumeOnWake lifts the hub pause with no origin reconcile, and the notice says only that the pause was lifted
    fix: "In carry (internal/hibernateengine/watch.go) when r.Phase is PhaseResuming give failAtWake a reason that names the skipped step, so the failed notice reads that the wake reconcile did not run and that lym cleanup --apply runs it. Add a case to the TestTakeUp row for a resuming takeover asserting the notice names lym cleanup --apply."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/hibernateengine/carrier.go:46-69"
    summary: after a cancel and a fresh submit the old sequence keeps the carrier busy until its next poll, and no event wakes dobby when it ends, so the new request can wait up to the loop timer
    fix: "Make the carrier's goroutine end wake dobby, or let Tick take up a request that is not the in-flight sequence's own. The smallest change is for Carrier.Tick, when running is true, to read the request and, if it is non-terminal and ours reports false (a new request written after a cancel), return a Deps-level wake that dobby's tick already honours, or have the goroutine touch the request file after it clears running so the watch event runs the next tick. Add a TestCarrier row that cancels, submits again while the first sequence still waits, and asserts the second request is carried once the first sequence has ended."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: "internal/cleanupcli/cleanup_integration_test.go:329"
    summary: a test comment still says the hibernate daemon paused the pair, though the same diff reworded the cleanupcli docs away from the daemon
    fix: "In internal/cleanupcli/cleanup_integration_test.go line 329 change the comment to say a pair the hibernate request's wake paused."
---

### [BLOCKING] hibernateEndedPhases repeats the engine's phase values as literals

**Location:** `internal/fabriccli/anchordeps.go:43-44`
**Issue:** The diff adds `var hibernateEndedPhases = []string{"done", "failed", "cancelled"}` to `fabriccli`, which restates the engine's terminal `Phase` values (`PhaseDone`, `PhaseFailed`, `PhaseCancelled` in `internal/hibernateengine/request.go`, whose `Terminal` method is the one definition).
`PATTERN-named-constants` asks for one named constant, declared once and referenced everywhere, never a repeated literal, and the constraints section makes every violation BLOCKING.
The commit carries no `Pattern-Departure:` trailer.
The file's own comment explains that `fabriccli` cannot import the hibernate packages, which is why `request.json` is already repeated there; that explains why the literals exist, but it does not make a second, untested copy of three wire values a single declaration.
A rename or a new terminal phase in the engine leaves `fabriccli` reading an ended request as pending with nothing to catch it (see F2, which has no test on this either).

### [MEDIUM] the ended-phase rule of hibernateDaemonLive has no test

**Location:** `internal/fabriccli/anchordeps.go:240-256`
**Issue:** `hibernateDaemonLive` now reads the request's `phase` and treats an ended request as not live, which the summary states as "an ended request no longer blocks it".
No test calls `hibernateDaemonLive`; `anchor_test.go:183` stubs `HibernateLive`.
If the phase check broke, an ended request that still names the running prime dobby would again refuse `lym fabric anchor --set` on the prime, with every test green.
This is a behavior the diff adds with no coverage.

### [LOW] the hibernate directory may not exist when the watch opens

**Location:** `internal/beamcli/dobbywatch.go:129-131`
**Issue:** The prime's watch set now names the request file's directory.
`openDobbyWatch` skips a directory that does not exist and tries it again only at the next event from another watcher, and nothing in the diff creates the hibernate directory at dobby start; the removed spawn made it only for its log.
On a hub that never hibernated, the first request creates the directory and wakes nothing, so an idle prime dobby takes it up on its timer, up to the one-minute idle ceiling (`internal/dobbyengine/loop.go:27`).
`internal/beamcli/doc.go` ("a written request wakes the tick that takes it up") and the summary's "within a second of the write" do not hold for that case.
The request is still taken up, so the cost is delay.
Found by the correctness and affected-set lenses and by the handler.

### [LOW] a takeover in the resuming phase lifts the pause without the reconcile

**Location:** `internal/hibernateengine/watch.go:67-70`
**Issue:** `carry` ends a request found in `resuming` as failed and, when `ResumeOnWake` is set, lifts the hub pause.
The origin reconcile and the pair pause entries the wake would have made never run, and the notice says only that the pause was lifted, so the hub ends unpaused and unreconciled with the operator not told.
The notice should name the skipped reconcile and `lym cleanup --apply`.

### [LOW] a cancelled sequence keeps the carrier busy and nothing wakes dobby when it ends

**Location:** `internal/hibernateengine/carrier.go:46-69`
**Issue:** After `Cancel` and a fresh `Submit`, the old goroutine runs until its next `stillOurs` check, up to the 30-second trigger poll ceiling.
`Tick` takes up nothing while `running` is set, the new request's write already fired its watch event, and the goroutine's end fires none, so the next take-up waits for the loop timer, up to a minute more.
The delay is bounded and no state is wrong.

### [NIT] a stale test comment names the hibernate daemon

**Location:** `internal/cleanupcli/cleanup_integration_test.go:329`
**Issue:** The comment reads "a pair the hibernate daemon paused", while the diff reworded the sibling `cleanupcli` doc and function comments to the request's wake.

## Rejected

- Lens code-tests, `withdrawn` has one covering row and its other callers have none (LOW): the helper has a pinning row, `watch_test.go:1075`, and the other callers are one-line delegations to it, reached with an entry already recorded, where `Cancel`'s own lift covers the entry; a per-caller row would test the helper again.
- Lens code-tests, the prime-only wrap `deps.CheckBinary = holdReexec(...)` is not asserted (LOW): `TestHoldReexec` pins the helper, and the assignment sits beside the `Tick` assignment the integration test already checks; asserting it needs a real `selfexec` check or a comparison of function values, so a test would pin the wiring line and nothing else.
- Lens code-correctness, a hibernate-wiring error from `NewDobbyCarrier` stops the prime's dobby from starting (LOW): `hubDeps` fails only on `PrimeName` or the prime's worktree resolution, which the dobby's own geometry in `hubDobbyDeps` already needs, so no new failure path exists, and the code-quality rule against defending internal wiring applies.
- Checked and clean, `--after-all` and `battenrecipe.PastLand` (focus bullet 3): `--after` still tests the run through `PastLand`, and `--after-all` still reads every run listed in the prime's runs directory, which holds every beam's batten run, and waits on each one in state running; `triggerReading` is unchanged by the diff, and the decision asks only that the request read the runs of every beam, so no finding.
- Checked and clean, the parent entry on `resume_on_wake`: `Submit` still records `SubmitRequest.ResumeOnWake` on the request, the sequence still lifts or keeps the owned entry on it after the wake, and the diff does not touch `internal/hibernateconfig` or the flag.
- Checked and clean, the pause scope (decision 4): the sequence runs in a goroutine apart from the tick steps a pause holds back, gate-held calls leave a waiting marker that the confirmation's network-operation reading never counts, and the wake reconcile passes the gate under the request's own exemption.
- Checked and clean, the identification of the prime's dobby (decision 1): the carrier is wired only where `geom.Slug` is empty in `hubDobbyDeps`, and a beam's dobby gets neither `Deps.Tick` nor the request file in its watch.
