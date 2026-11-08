---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/creelcli/spawn.go:61
    summary: "PATTERN-child-processes now says a spawned lym command names its binary via selfexec.LymBinary, but several lym spawns still call os.Executable directly and the change description does not list them as exempt"
    fix: "Convert each remaining lym spawn or pin to selfexec.LymBinary, or to selfexec.ResolveLymBinary over its injectable lookup, in internal/creelcli/spawn.go:61 (detached creel daemon), internal/gatecli/detachfront.go:158 (gate run-detached), internal/reedcli/remove.go:63 (detached reed remove), internal/reedcli/window.go:83 (lymPath handed to the terminal opener) and internal/boardengine/spawn.go:32 (syncExecutable default, a test still replaces it). Keep each existing testing.Testing or suppress guard. If the author decides a site that already refuses under go test may stay on os.Executable, amend the PATTERN-child-processes bullet and the PATTERN.md entry text to name that exemption, and the resolve-once exemption of loomcli drivewiring.go:171 and battencli wire.go:1039, so the rule matches the code. The vscode config stamp in internal/ideengine/spawn.go:225 is a path written for a later VS Code task, so convert it too or name it in the same exemption."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/reedengine/buttonclick.go:156
    summary: "The button prompt re-entry takes its binary from deps.executablePath raw, so an injected os.Executable reaches a tmux run-shell line, and the new reedengine/doc.go sentence claiming no pin names the test binary even then is false"
    fix: "In internal/reedengine/buttonclick.go:156 replace deps.executablePath() with selfexec.ResolveLymBinary(deps.executablePath), keeping the existing error branch. In internal/reedengine/doc.go around lines 399-402 keep the sentence that every pin goes through ResolveLymBinary only once that holds, and state that SpawnWatchdog calls selfexec.LymBinary directly because it takes no injectable lookup."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun.go:106
    summary: "No test pins that a tripped process cap fails the gate: the forced exit 1, the capLine early return, the no-rerun clause in failedOnShortage and the marker and samplePeriod guard are all untested"
    fix: "Add a case to the existing tier-run tests in internal/gatecli that runs runTier or runInvocations with a stand-in cap trip, through a seam for newProcessCap such as a processCap factory field on testRequest. Assert that a command exiting 0 returns exit 1, that tierResult.processCapped is set, that failedOnShortage returns the empty string for that result, and that the cap line reaches request.stderr and the causes. Add a row where env carries no run marker and assert the cap does not start."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxkit/tmuxkit.go:111
    summary: "A refused re-exec exits 2 after HermeticGitEnv has already made the process run directory, and the exit skips the removal Main performs after the run, so each refused process leaks one directory"
    fix: "In tmuxkit.Main, before returning refusedReexecExit, remove the process run directory the way Main does after m.Run, so a refused process leaves nothing behind. Alternatively give HermeticGitEnv no run-directory side effect before Main. Cover it by making the integration test in internal/testkit/tmuxkit/reexecguard_integration_test.go point the child at a private temp root and assert nothing is left there."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/bindings_test.go:260
    summary: "The test-binary row asserts only the toggle and button lines, so the switch and follow bindings, the window command, the pane prelude, loom's loopExecutable and fronttask's lookup error have no row showing they refuse"
    fix: "Extend the TestBinaryExecutable row in internal/reedengine/bindings_test.go to assert that the prev, next and follow lines are also absent, and add rows for composePaneLaunchLine and composeWindowCommand handed os.Executable that assert a dropped prelude. Add a row to the loomcli test of loopExecutable where the lookup fails and assert the empty executable, and a row to fronttask TestStartAction where the lookup fails and assert the locate-the-lym-executable error."
---

### [BLOCKING] Remaining lym spawns bypass selfexec.LymBinary

**Location:** internal/creelcli/spawn.go:61; internal/gatecli/detachfront.go:158; internal/reedcli/remove.go:63; internal/reedcli/window.go:83; internal/boardengine/spawn.go:32; internal/ideengine/spawn.go:225
**Issue:** The diff adds to `PATTERN-child-processes` that a command naming this lym binary for a pane, tmux job, binding, hook or a lym child or detached process resolves it through `selfexec.LymBinary`, and that only a read of the binary itself calls `os.Executable` directly.
Each listed site starts a detached lym child or hands a lym path to another process, and still calls `os.Executable`.
Most are guarded by `testing.Testing()` or a suppress flag, so they cannot run a test binary today, but the rule text states no such exemption, and the change description's "Sites left on `os.Executable`" list names none of them.
The decision record's item 2 asks that every such pin use the helper.
The constraint outranks the fact that the code works.
Reed's `watchdog.go` stat of its own binary and the build-identity reads are reads of the binary itself and are not flagged.

### [BLOCKING] Button prompt re-entry resolves the binary raw, and the reedengine doc overclaims

**Location:** internal/reedengine/buttonclick.go:156; internal/reedengine/doc.go:399
**Issue:** `deps.executablePath` defaults to `selfexec.LymBinary`, but a test may inject any lookup, and every other reed pin wraps its injectable lookup in `ResolveLymBinary`.
The button prompt builds a `run-shell -b` line from the raw result, so an injected `os.Executable` yields a line that runs the test binary, against the PATTERN bullet's "or `selfexec.ResolveLymBinary` over an injectable lookup".
The new doc text says no prelude, binding, button or Slack line names the test binary "even when a test hands the engine os.Executable", which this path contradicts.
`SpawnWatchdog` is compliant, since it calls `selfexec.LymBinary` directly, but the same sentence's "every pin passes that lookup through ResolveLymBinary" does not describe it.

### [MEDIUM] The process cap's failure path is untested

**Location:** internal/gatecli/tierrun.go:106; internal/gatecli/tierrun.go:183; internal/gatecli/tierrun.go:316
**Issue:** The decision record's item 4 is that a runaway loop fails the gate.
`processcap_test.go` covers `processCap.run` and `runMarker` in isolation, but nothing covers the wiring in `runTier` and `runInvocations`: the forced exit 1 when the command exits 0, the `processCapped` flag, the no-rerun clause in `failedOnShortage`, the early return on `capLine`, and the `marker != "" && samplePeriod > 0` start condition.
`newProcessCap` itself is production wiring, and a seam over it is the cheapest way to reach the rest.
A change that dropped the forced exit would pass every current test.

### [LOW] A refused re-exec leaks the process run directory

**Location:** internal/testkit/tmuxkit/tmuxkit.go:111; internal/gitkit/hermetic.go:25
**Issue:** The removed guard ran at the top of `HermeticGitEnv`, before any directory existed.
`TestMain` now calls `HermeticGitEnv` first, which creates the neutral git config under the process run directory, and then `Main` refuses and returns 2 before the cleanup it runs after `m.Run`.
Each refused process leaves one directory, which stays small per process but accumulates in the runaway the guard exists for.

### [LOW] Refusal rows missing for the sibling reed lines and two lookups

**Location:** internal/reedengine/bindings_test.go:260; internal/loomcli/looparm.go:71; internal/fronttask/hub.go:131
**Issue:** The `TestBinaryExecutable` row pins the toggle, state and button lines.
The switch and follow bindings, the window command and the pane prelude take the same `ResolveLymBinary` call but have no row, so a regression to the raw lookup passes.
`loopExecutable`'s refusal and `startAction`'s lookup error have no row either.

## Rejected

- code-correctness #2, a cap shared across concurrent tiers: the fork did not confirm that tiers run concurrently, `runTier` is called in sequence from the gate's tier loop, and the cap exists to end the whole run's processes.
- code-correctness #3, `loopExecutable` returning an empty executable under `go test`: the fork judged it safe and equal to the old behaviour on a lookup error.
- code-correctness, a symlinked stand-in passing `ResolveLymBinary`: only a test that injects such a stand-in reaches it, and `tmuxkit.Main` backstops it; not a defect.
- code-affected-set #3, the resolve-once exemptions in `loomcli/drivewiring.go` and `battencli/wire.go`: folded into F1's fix, which names them in the amended rule.
- code-tests, `proc.PIDsWithEnv` error paths and the Windows stub, and the unresolved-self branch of `ResolveLymBinary`: a read of `/proc` and a one-line stub, with no behaviour worth a row.
- code-tests, `newProcessCap` production wiring and the per-caller swaps in adopt, orch, batten, fabric and drift: their `testing.Testing()` guards return before the lookup, so a row cannot reach them.
- Audit of the gitkit deletion: no live reference remains; the guard's coverage moved to `tmuxkit` and `cmd/lym/tmuxisolation_test.go` enforces `Main` in every package with an empty allowlist.
- Item 3 as one guard in `tmuxkit.Main` with one proving test: the decision record asks for the guard in every package, and the single call all packages already make covers it by construction.
