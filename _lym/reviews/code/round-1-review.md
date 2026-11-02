---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/config.go:391
    summary: The diff adds a third bare "card" gate-key literal to decodeGates, against PATTERN-named-constants
    fix: In internal/gateslot/tiers.go declare one constant for the card gate key name, for example cardGateKey = "card", beside scopeCard. Replace the "card" literals in internal/gateslot/config.go at the scopeCard check (line 378), the new fixer check (line 391) and the retired-list migration (line 568) with it, so the key name is declared once and referenced everywhere.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatefixer/rowgate.go:117
    summary: waitOnly marks the fixer record terminal after a passing earlier entry, so a restarted Darn-Write run loses the live marker and the fixer's reply before the verify entry judges
    fix: In rowgate.go give waitThenRun a flag that says whether a passing result marks the record terminal. waitOnly passes false, so an earlier entry that passes leaves the record live and only an entry that fails over the finished tree marks it, with the reply or dead note appended as today. The fixer entry's spawning path and its Final closure pass true, so the fixer entry marks the record once its own closure returns. A later entry that waits on the already-ended fixer returns at once, since its reply file exists or its strand is gone. Add rows to TestWrapRowGate in rowgate_test.go for a clean entry that passes over a live record then a verify entry that fails (the findings carry the reply, the record is terminal after verify returns, no second fixer spawns), and for a restart between the two (the record is still live, so Expired reads true).
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatefixer/rowgate.go:204
    summary: waitThenRun marks the record terminal even when the inner closure returned an error, so no result judged over the finished tree exists yet
    fix: In waitThenRun return the inner closure's error before calling markTerminal, leaving the record live, so the next evaluation of the visit waits on the ended fixer again and judges. Mark terminal only after inner returned without error. Add a row to TestWrapRowGate in rowgate_test.go where the inner closure errors after a wait and the record stays live.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/detachfixer.go:112
    summary: runWithFixer hands every non-zero first exit to a fixer, including a refusal that writes its envelope to stdout and nothing to stderr, which D4 does not call a red verdict
    fix: In runWithFixer, before calling fixAndRerun, return first when the run's stderr file is absent or empty, since a gate verdict writes its failure report to stderr and a refusal writes a JSON envelope to stdout only. Add a row to TestRunWithFixer in cardfixer_test.go where the first run exits 1 with an empty stderr and no fixer is spawned.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/detachfixer.go:206
    summary: The card runner marks the fixer record terminal before the re-run starts, where D1 says a card caller marks it once it holds a result judged over the fixer's finished tree
    fix: In fixAndRerun move the store.markTerminal call, with its Warn on failure, to after d.runRequest returns the re-run's exit, and mark it on every path that reaches the re-run, an abandoned one included. Add an assertion to TestRunWithFixer/RedCallIsFixedAndRerun in cardfixer_test.go that the key directory's fixer record is still live while the re-run runs and terminal once the call has ended.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/detachfront.go:830
    summary: No test pins that writeNewRun writes the caller's role into the run request, the field the fixer-role guard reads, so dropping it would let a fixer's own gate call hand off
    fix: Add a case to detachfront_test.go that builds a frontCall for the strand lym:work:gate-fixer-2, calls writeNewRun, reads request.json back as a detachRequest and asserts its Role is gate-fixer-2, and a second row for a task strand asserting the role of that strand.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/drivewiring.go:281
    summary: No test pins that the three goto sites wire fixerEndingGuard with endRunFixers, which D8's goto-leaves-no-fixer criterion depends on
    fix: Add a test in internal/loomcli, beside wiring_seams_test.go, that runs the guard each of drivewiring.go:281, arm.go:464 and arbiterconsume.go:28 builds, with an admitting inner guard and a recording end seam, and asserts every site ends each role of gatefixer.Roles() before the move. If the sites reach endRunFixers only as a method value, give loomCLI one field for the end function that defaults to endRunFixers, so the test can substitute a recorder.
  - id: F8
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/gatefixer.go:209
    summary: gateFixerSeam refuses a non-positive fixer timeout, a state decodeConfig already refuses at config load
    fix: Delete the FixerTimeout() <= 0 branch in gateFixerSeam and its row in TestGateFixerSeam in internal/loomcli/gatefixer_test.go.
  - id: F9
    severity: NIT
    class: consistency
    origin: handler
    location: internal/loomcli/gatefixer.go:94
    summary: gateFixerBodies keys the Darn-Write entry with a bare "verify" literal where the Plan-Write and Discussion-Write keys use the named constants planGateEntry and discussionGateEntry
    fix: In internal/loomcli/gatefixer.go declare a constant for the Darn-Write fixer entry name, for example darnGateEntry = "verify", in the const block that holds planGateEntry and discussionGateEntry, and use it as the gateFixerBodies key.
---

### [BLOCKING] A third bare "card" gate-key literal in decodeGates

**Location:** internal/gateslot/config.go:391
**Issue:** PATTERN-named-constants asks for one named constant for a name that can go stale, referenced everywhere and never a repeated literal.
The diff's fixer check adds `name != "card"` beside the existing literals at lines 378 and 568 of the same file, so the key name now stands three times as a bare string, while the diff's own `darnWriteGateKey` shows the constant form.
Listed constraints bind as BLOCKING however small the violation.

### [MEDIUM] waitOnly marks the record terminal on a passing earlier entry

**Location:** internal/gatefixer/rowgate.go:117
**Issue:** D2 says the record is marked terminal by the first entry that fails over the finished tree, or by the fixer entry once its closure returns.
`waitOnly` calls `waitThenRun`, which marks the record terminal after the inner closure whether it passed or failed.
On Darn-Write a driver restarted while the fixer works finalizes, `clean` waits first and passes over the fixer's committed tree, and the record turns terminal.
The `verify` entry then finds no live record, so a failure there carries no fixer reply, which D2 and the acceptance criterion on `clean` findings require.
A second restart during the `verify` run also finds only a terminal record, so `Expired` reads false and the restart-during-re-run guarantee of D1 is lost.

### [LOW] waitThenRun marks the record terminal over an inner error

**Location:** internal/gatefixer/rowgate.go:204
**Issue:** `waitThenRun` calls `markTerminal` whether or not the inner closure returned an error, then returns the joined error.
D1 marks a record terminal once the caller holds a result judged over the finished tree; an erroring closure holds none.
The next evaluation of the visit finds no live record, never re-waits and, with the count at the cap, may spawn no fixer either.

### [LOW] runWithFixer hands a refusal to a fixer

**Location:** internal/gatecli/detachfixer.go:112
**Issue:** `runWithFixer` calls `fixAndRerun` for any first exit that is non-zero and not abandoned.
The exit-code section of `internal/gatecli/doc.go` gives exit 1 to a refusal, with its JSON envelope on stdout only, so a refused call on a key with a `fixer` spawns a cold fixer over an empty failure report and then re-runs into the same refusal.
D4 hands only a red verdict to the fixer.

### [LOW] The card runner marks the fixer terminal before the re-run

**Location:** internal/gatecli/detachfixer.go:206
**Issue:** D1 says a row or card caller marks the record terminal once it holds a result judged over the fixer's finished tree, so a restart during the re-run still finds it live.
`fixAndRerun` marks it before `rewriteTreeIdentity` and the re-run.
The fixer has already ended, so no wrong behaviour follows, but the code departs from the decision's stated order.

### [LOW] writeNewRun's role field has no test

**Location:** internal/gatecli/detachfront.go:830
**Issue:** The fixer-role guard in `handsOff` reads `request.Role`, which only `writeNewRun` writes from the front's strand name.
`TestCallKeyDir` pins the role parse and the key, and `TestHandsOff` and `TestRunWithFixer` build the request by hand, so dropping `Role: f.call.role` breaks no test while letting a fixer's own gate call hand off.

### [LOW] The goto-site wiring of fixerEndingGuard has no test

**Location:** internal/loomcli/drivewiring.go:281
**Issue:** `TestFixerEndingGuard` and `TestEndGateFixers` pin the guard in isolation.
Nothing pins that the three goto sites, `drivewiring.go:281`, `arm.go:464` and `arbiterconsume.go:28`, wrap their inner guard with it, so removing one leaves a live fixer editing under the next row, which the D8 acceptance criterion on a goto past a row forbids.

### [NIT] Unreachable non-positive fixer timeout branch

**Location:** internal/loomcli/gatefixer.go:209
**Issue:** `decodeConfig` refuses `fixer_timeout_min` below 1 at load, so a loaded `gateslot.Config` never reaches this branch.

### [NIT] Bare "verify" key beside named entry constants

**Location:** internal/loomcli/gatefixer.go:94
**Issue:** `planGateEntry` and `discussionGateEntry` are constants and the Darn-Write key in the same map is a literal.

## Rejected

- Lens code-correctness finding 4, an unreadable fixer record in `settleLiveFixer` runs the gate anyway: the record is the only source of the strand to stop, so with it unreadable there is nothing to attach to or stop, and D4 names no other action.
- Lens code-correctness finding 5, `spawnAndJudgeAgain` returns the failure standing when `SpawnOrAttach` errors: the decision record does not settle a failed spawn at a row, and the repeats are bounded by the writer's attempts since each failing re-prompt spends one.
- Lens code-affected-set finding 1, `docs/overview.md` is not updated: no rule of the repository, PATTERN.md or the plan names that file for this kind of change, and the gate module's own `internal/gatecli/doc.go` is updated.
- Handler check of the declared PATTERN change: `PATTERN.md` carries `PATTERN-gate-fixer` with exactly the declared text and the declaration names no background file, so nothing is raised.
- Handler check of the commit messages: no `Pattern-Departure:` trailer appears and the plan cards map one to one to the commits, so nothing is raised.
