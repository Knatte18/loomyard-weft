---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/verifytree/reach.go:160
    summary: >-
      a reach turn overwrites the listed file with only the current pending functions, so a function listed once can bring a second turn
    fix: >-
      In ReachTurn.findings, write the union of the old listed.Functions and pending.Functions to ListedPath, taking the new hash for an id whose hash changed, instead of writing only pending.Functions. Add a TestWithReachTurn row where turn 1 lists A, turn 2 lists only B, and a third arrival with A unreached again at the same hash gets no turn.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/triage.go:306
    summary: >-
      the prime's no-change-set branch of addTriage, required by D7 and the added entry on triage in the prime, has no test that drives it
    fix: >-
      Add a case to TestGateTest_Scenario in internal/gatecli/test_integration_test.go that fails a mapped test from the prime and asserts its first line under the test reads triage, no change set exists, no parent branch, map commit and the commit, with no lym gate reach --test line after it.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/loomcli/wiring.go:826
    summary: >-
      nothing asserts that wire sets weaverGeom.ReachPending and Env.VerifyReach, and both are nil-legal in envkit, so a dropped assignment silently turns off D3's reach turn
    fix: >-
      In internal/loomcli/wiring_test.go TestWire_ReviewKeysReachTheEnv, assert that c.env.VerifyReach is non-nil and that the weaver geometry's ReachPending is non-nil after wire, as the test does for ReachReport.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/gatefixer/rowgate.go
    summary: >-
      the live-record path of the row wrap runs the inner gate without the entry's re-run mark, so after a driver restart the judging run counts a failure in the reach turn
    fix: >-
      In rowWrap.spawning, change the live-record call to w.waitThenRun(store, live, func() (shuttleengine.GateResult, error) { return w.rerun.Run(inner) }, true), as spawnAndJudgeAgain does. Add a row to TestWrapRowGate that starts the wrap with a live record and asserts the inner call sees the mark on.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: internal/gatecli/reach.go
    summary: >-
      refreshMaps returns after a failed HeadSHA read with an error line only, so the note names no reason for any unrefreshed package
    fix: >-
      In reachStep.refreshMaps, on a failed run.fabric.HeadSHA(), call unrefreshed(pkg, a reason saying HEAD could not be read as its error line says, with the way forward to run the gate again) for each package before returning.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/testreach/changedlines.go:386
    summary: >-
      the LCS table is quadratic in the size of the changed middle, so a rewritten large file allocates hundreds of millions of ints in lym gate mutate
    fix: >-
      In ChangedLines, when len(baseMiddle)*len(headMiddle) passes a named constant cap, skip the table and mark every line of headMiddle as changed. Add a ChangedLines row over a middle past the cap.
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/testcover/linemutant_test.go:295
    summary: >-
      TestPlanLineMutants pins no row for a deleted && or || operand in the operator order, a logger import with an alias, a local that shadows the logger package name, or a shifted rune constant
    fix: >-
      Extend lineMutantSource and TestPlanLineMutants with a line holding an && condition and a constant, a logger import under an alias, a local named like the logger package, and a rune constant; assert the operator chosen per line and that the aliased logger call gets none while the shadowing local's call does.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/gatecli/mutate_integration_test.go:614
    summary: >-
      TestGateMutate has no row for the note of a diff that changes no function, the prime's stale-shared-map and no-map reasons, or the unbuildable-mutant count
    fix: >-
      Add rows to TestGateMutate: a --base equal to HEAD asserting the changes-no-function note and no mutant; a prime call with --base whose shared map is stale, and one with no map, asserting reasonSharedMapStale and reasonNoMapToMutate in not_judged; and a landing whose mutant the compiler refuses, asserting the unbuildable count.
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: design
    location: internal/verifytree/reach_test.go:235
    summary: >-
      no TestWithReachTurn row shows that a Pending or Terminal inner failure adds nothing to the failing count
    fix: >-
      Add a row to TestWithReachTurn where the inner gate returns a Pending result, then a Terminal result, then a pass with a pending report at the budget edge, and assert the failing count stayed unchanged by the turn running.
    origin: lens:code-tests
  - id: F10
    severity: LOW
    class: design
    location: internal/gatecli/reachverb_integration_test.go:530
    summary: >-
      no TestGateReach row covers the root-commit skip, where a walked commit has no parent
    fix: >-
      Add a TestGateReach row whose --since range includes the repository's root commit, and assert the verb exits 0 and lists no landing for it.
    origin: lens:code-tests
  - id: F11
    severity: LOW
    class: design
    location: internal/testreach/index.go:785
    summary: >-
      readIndex takes a read lock on a lock file beside the index, which creates that file in the shared maps directory for every reader, against D1 keeping that directory written from the prime alone and D6's lym gate reach writing nothing
    fix: >-
      In readIndex, read the index with a plain os.ReadFile and json decode instead of state.ReadJSON, since WriteIndex replaces the file atomically; or state in the package doc that a reader leaves an index lock file there. Extend the TestGateReach unchanged-directory check to run with an index present.
    origin: handler
---

### [MEDIUM] A reach turn forgets functions it listed before

**Location:** `internal/verifytree/reach.go:160`
**Issue:** `ReachTurn.findings` writes `reachListed{Functions: pending.Functions}` after a turn, replacing the earlier set.
D3 says a function listed once never brings another turn.
With turn 1 listing {A} and a later turn listing {B}, A is dropped, so A coming back unreached at the same hash brings a second turn.

### [MEDIUM] The prime's no-change-set triage branch is untested

**Location:** `internal/gatecli/triage.go:306`
**Issue:** D7 and the parent's added entry make the prime answer each mapped failing test with the no-change-set line naming the missing parent branch, and no command.
`TestTriageLookup_Lines` drives the lookup with a no-parent callback and the scenario test pins only the unmapped line in a worktree.
Nothing drives `addTriage` in the prime, so the closure could return HEAD as the base unnoticed.

### [MEDIUM] The reach turn's seam wiring is unasserted

**Location:** `internal/loomcli/wiring.go:826`
**Issue:** `wire` sets `weaverGeom.ReachPending` and `Env.VerifyReach`, the two channels D3's reach turn reaches the loomshed verify gate and the weaver batch gate through.
`envkit` allows both nil, and `TestWire_ReviewKeysReachTheEnv` asserts only `ReachReport`, so a dropped assignment turns the turn off with no failing test.

### [LOW] The live-record path runs the gate without the re-run mark

**Location:** `internal/gatefixer/rowgate.go`
**Issue:** After a driver restart finds a live fixer, `spawning` calls `waitThenRun` with `inner` itself.
D3 marks the run after a fixer as a re-run, so this run counts a failure in `WithReachTurn` that the rule says it must not.

### [LOW] A failed HEAD read leaves unrefreshed packages with no reason

**Location:** `internal/gatecli/reach.go`
**Issue:** `refreshMaps` returns after `run.fabric.HeadSHA()` fails with one Errors line.
D3 says the note names each package left unrefreshed and why, and here it names none.

### [LOW] The line diff is quadratic in the changed middle

**Location:** `internal/testreach/changedlines.go:386`
**Issue:** The LCS table has one cell per pair of base and head middle lines, so a rewritten file of ten thousand lines allocates about a hundred million ints for `lym gate mutate`.

### [LOW] Line mutant operator rows are missing

**Location:** `internal/testcover/linemutant_test.go:295`
**Issue:** D9's operator order is pinned for a negated condition over a constant and for a dropped statement.
No row covers a deleted `&&` or `||` operand, a logger import under an alias, a local that shadows the logger package name, or a rune constant.

### [LOW] `lym gate mutate` rows are missing

**Location:** `internal/gatecli/mutate_integration_test.go:614`
**Issue:** No row covers the note of a diff that changes no function, `reasonSharedMapStale` and `reasonNoMapToMutate` in the prime, or the unbuildable-mutant count.

### [LOW] Pending and terminal inner failures are unpinned in the turn

**Location:** `internal/verifytree/reach_test.go:235`
**Issue:** `WithReachTurn` leaves a Pending or Terminal inner failure out of the failing count, and no row shows it.

### [LOW] The root-commit skip is unpinned

**Location:** `internal/gatecli/reachverb_integration_test.go:530`
**Issue:** `runReachTest` skips a commit with an empty parent, and every range in `TestGateReach` starts at a non-root seed.

### [LOW] Readers create a lock file beside the shared index

**Location:** `internal/testreach/index.go:785`
**Issue:** `readIndex` reads through `state.ReadJSON`, whose read lock creates a lock file beside the index when an index exists.
D1 keeps the shared maps directory written from the prime alone, and D6 says `lym gate reach` writes nothing.
`TestGateReach` checks the directory unchanged only while no index exists, so it does not see this.

## Rejected

- Scope lens F1, the D11 board note not shown in the diff: the board holds note R1, "Run the mutation trial with lym gate mutate", whose body asks for the trial and for the decision on building the `mutation` report step and listing it in the template's diff keys.
- Scope lens F2, `pattern/PATTERN-gate-slot.md` changed without a declared entry: the edit is forced by D3's refresh holding a gate slot, and the record's constraints section names `PATTERN-gate-slot` for it.
- Affected-set lens: reported no finding, and I found none to add.
- The nil-location path of `reachStep.run` for a call outside a hub: it reads the report path before its switch exactly as the redundancy step does, so it is no new defect of this diff.
