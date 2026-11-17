---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/gatecli/testmaptree.go:79
    origin: lens:code-correctness
    summary: dirtyOwnedFile matches records-side paths from Fabric.Status against code package ownership, so a records edit can mark a module-root package dirty
    fix: In dirtyOwnedFile, read only the code side, by calling r.fabric.CodeWorktreeChanges() as dirtyModuleFile does, or by keeping only the Status entries whose Side is fabricengine.SideWarp, then match those paths against trees. Add a row to the D6 subtests of TestGateTestMapLinkedPackages with a records-side edit that leaves a package measured.
  - id: F2
    severity: LOW
    class: scope
    location: internal/gatecli/cli.go:78
    origin: lens:code-scope
    summary: The testCmd help text drops two rules and two examples that no decision asks to change
    fix: In the Long text of testCmd in internal/gatecli/cli.go, restore the sentences "Inside a slot already held by an enclosing gate run it runs without acquiring another." and "Each tier prints a summary line to stderr and writes its record and log under the gate directory.", and the examples "lym gate test --wait --gate finalize" and "lym gate test --upto 2", beside the new --map line and example.
  - id: F3
    severity: NIT
    class: scope
    location: internal/gatecli/run.go:353
    origin: lens:code-scope
    summary: The --map refusal list also names --tags, which D1's list omits, and the generic --tags refusal already covers it
    fix: Remove the "--tags" entry from the flag list in the mapMode branch of flagShapeRefusal in run.go, the --tags row of TestGateTest_RefusesBeforeResolvingTheHub in cli_test.go, and the `--tags` mentions in the --map help line of cli.go and the Coverage maps section of doc.go, so the generic --tags refusal handles it.
---

### [LOW] Status-based dirty check reads the records side

**Location:** internal/gatecli/testmaptree.go:79
**Issue:** `dirtyOwnedFile` reads `Fabric.Status()`, which merges the warp and weft sides, and tests every path against the code packages' ownership. A weft path is a records-repository path.
A package at the module root owns every path outside its nested package directories, so any uncommitted records edit makes it look dirty, and D6 says a package is dirty only for a file the package or a linked package owns.
`dirtyModuleFile` already restricts itself to the code side, so the two D6 checks disagree.
The code branch passed in the lens's step-by-step run; this path is reachable only for a module-root package or a records path that collides with a package path.

### [LOW] Help text loses rules and examples outside the decision

**Location:** internal/gatecli/cli.go:78
**Issue:** The hunk that adds the `--map` line also deletes two sentences and two examples of the plain `lym gate test` help.
The sentences state the enclosing-slot rule and where each tier's summary and record go.
The examples cover `--gate finalize` and `--upto 2`.
D1 licenses a `--map` line and example only, and the removed text stays true for plain calls.
No `Pattern-Departure:` trailer in the range explains it.

### [NIT] `--tags` added to the `--map` refusal list

**Location:** internal/gatecli/run.go:353
**Issue:** D1 lists the refused flags and `--tags` is not among them.
A plain `--tags` call is already refused whatever the mode, so the added entry changes only which message names it.
It also adds a test row, and a `doc.go` and help-line mention, for that message.

## Rejected

- code-tests F1 (no map-run test sets a `tier_parallel` entry): the override is applied by gateslot's config load into each `Tier.Parallel` (`applyTierParallel`, `internal/gateslot/config.go:326`), which the diff does not change, so the diff adds no behaviour there that needs a map-run row.
- code-tests F2 (no test that the detached key differs for a map call): the detached call's key is built from `request.words()` (`internal/gatecli/detachfront.go:273`), and the `MapSet` and `MapUnset` rows of `TestTestRequestWords_MainPackages` pin `--map` in those words, so the key row is covered.
- code-scope F1 and the `--tags` part of code-affected-set's coverage list are kept as F3 above, at NIT: the generic refusal makes the added branch harmless.
- code-affected-set F1 duplicates code-scope F2 and is kept once, as F2.

## Handler checks, no finding

- The stale-name grep for `gate testmap`, `testmapwait`, `testmapaudit`, `TryAcquire(`, `FailedAt`, `ProfiledFuncs`, `stampDifferences` and `GoRoot(` outside `docs/research/` finds no live reference to a retired symbol.
- `PATTERN-gate-slot` names the map run as the third place, `PATTERN-daemon-wakeups` and the `cmd/lym/daemonwakeups_test.go` row go together, and `PATTERN-test-speed` rewrites its coverage-map section.
- `mapSlotAsk` treats `Slots == 0` as every slot, matching `Tier.Slots` ("0 means every slot", `internal/gateslot/tiers.go:42`).
- `gateEnv` strips an inherited `LYM_PREBUILT_LYM` through `gateslot.StripPrebuilt`, so map mode builds no `lym` and its children see no prebuilt binary.
- The redundancy report lists the changed tests as unique or not judged, so no test-economy candidate arises from it.
