---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/machinecli/start.go:267
    summary: Starting an owned beam beneath an unowned prime still claims the prime through child preparation.
    fix: Carry an explicit no-prime-claim option from every machine-start child through beamcli into orchcli preparation and dobby startup. Suppress the preparation claim and prevent EnsureLive from claiming a prime under that option, while preserving ordinary beam-start behavior. Extend the unowned-prime case across the real child wiring and assert that prime_owner remains empty after the owned beam starts.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/machinecli/start.go:168
    summary: Normal terminal output hides attach commands, skipped-owner details and standing hold information.
    fix: Give machine start terminal presentation that emits each worktree outcome and detail, every fallback attach command, and the hold machine, subjects and own-machine rerun without requiring --envelope. Preserve the JSON contract for scripts and exercise the command through the readable-output wrapper in its existing output tests.
    origin: lens:code-affected-set
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/dobbyengine/launcher_test.go:100
    summary: The contained catch-up test needs the required keep annotation for its distinct removal assertion.
    fix: Keep TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing and add //testtiming:keep directly above its declaration, with a reason stating that it proves removal of a pre-existing launcher after ownership loss, which the startup test does not assert.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/machinecli/start.go:277
    summary: A failed dobby is reported as started because the child decoder discards outcome, detail and warnings.
    fix: Decode the beam-start outcome, detail and warnings in beamStartEntry and preserve them in the machine-start result. Treat failed or refused dobby outcomes as unsuccessful worktree starts even when the orch succeeded and the entry error is empty. Retain warnings for intentional orch-only starts and add the failed-dobby case to the command test table.
    origin: lens:code-correctness
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/desktoplauncher/desktoplauncher.go:67
    summary: Concurrent launcher additions and removals can leave a hub launcher without the all-hubs launcher.
    fix: Serialize Desktop.Add and Desktop.Remove across processes with one lock shared by every hub using that desktop. Hold it across both Add writes and across removal, Shortnames and all-hubs deletion. Add coordinated lifecycle coverage showing that an add cannot interleave between the empty-desktop check and all-hubs deletion.
    origin: lens:code-correctness
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/machinecli/start_test.go:130
    summary: Every machine-start test bypasses the new subprocess and envelope-decoding boundary.
    fix: Add a controlled-child tier1 command-boundary test using the production beamStart adapter. Assert cwd and arguments, successful and partial-result envelopes, refusal without a result, malformed output and execution failure through the command result. Test this as one boundary behavior, not separate helper tests.
    origin: lens:code-tests
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/machinecli/start_test.go:149
    summary: Window assertions count calls without verifying the workspace or worktree each call opens.
    fix: Record terminal directory, title and executable in startCalls and assert their exact expected values, plus the exact workspace path, in the existing ownership table. Include the prime and an owned beam so passing the prime directory for both fails.
    origin: lens:code-tests
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/machinecli/start_test.go:193
    summary: Failure isolation and window-opening recovery have no behavior coverage.
    fix: Extend the existing command tables with a failed hub followed by a successful hub, mixed successful and failed child entries, workspace-open failure and terminal-open failure. Assert continuation, aggregate exit status, windows only for successful entries, workspace warnings and the attach command after a terminal-open error.
    origin: lens:code-tests
  - id: F9
    severity: MEDIUM
    class: design
    location: internal/desktoplauncher/locate_test.go:52
    summary: Discovery tests replace both directory resolvers and leave the new filesystem and WSL query behavior untested.
    fix: Exercise Locate with isolated environment and config files for configured desktop selection, missing-config fallback and config-read failure. Add controlled query-boundary coverage for PowerShell desktop output, wslpath conversion and query failure or empty output, using tier1 only where a subprocess is needed.
    origin: lens:code-tests
---

### [BLOCKING] The beam child claims an unowned prime

**Location:** internal/machinecli/start.go:267

**Issue:** With an empty prime owner and an owned `pay` beam, the plan skips the prime and starts `beam start pay`.
`internal/beamcli/start.go:24` prepares the first target of every child.
That reaches `orchcli.prepare`, whose prime-claim call is at `internal/orchcli/start.go:495`.
The production seam at `internal/orchcli/cli.go:281` calls `claimPrimeFor` for beam locations too.
`internal/orchcli/ownerpause.go:104` claims the hub's unowned prime through `boardengine.SetOwners`.
This violates D10's no-claim bound and its unowned-prime acceptance criterion.
The existing test substitutes already-decoded child entries, so it does not reach this side effect.

### [BLOCKING] The operator cannot see required start details

**Location:** internal/machinecli/start.go:168

**Issue:** The root wraps operator commands with readable output on a terminal at `cmd/lym/main.go:180`.
The new result nests worktrees and holds inside the `start.hubs` object list.
`internal/output/summary.go` renders each hub's plain fields and only descendant error, warning and way-forward keys.
It omits the nested attach command, skipped-owner detail, hold machine, subjects and rerun.
Thus desktop launches and ordinary terminal invocations omit information D10 explicitly requires them to print.
The generic footer suggesting `--envelope` does not satisfy those first-invocation requirements.

### [BLOCKING] Preserve and mark the distinct catch-up assertion

**Location:** internal/dobbyengine/launcher_test.go:100

**Issue:** The supplied current redundancy report lists this test as contained by the startup launcher test.
The catch-up test creates a launcher before catch-up and asserts its subsequent removal.
The startup test's foreign-owner case begins without a launcher, so it cannot prove that removal.
The assertion merits retention, but `PATTERN-test-speed` and the rubric require a reasoned keep annotation.
No annotation precedes this test.

### [MEDIUM] A failed dobby becomes a successful machine start

**Location:** internal/machinecli/start.go:277

**Issue:** `orchcli.startDobbyStep` records a failed spawn in the dobby outcome, detail and warnings.
It lets the orch start continue, which can succeed without an entry-level error.
`beamcli.startBeams` preserves those fields, but the new decoder retains only worktree and error.
The machine command consequently emits `started`, opens its window and succeeds with no failure explanation.
The dobby needed to resume the work is absent.

### [MEDIUM] Shared launcher upkeep races between processes

**Location:** internal/desktoplauncher/desktoplauncher.go:67

**Issue:** Hub A removes its last launcher and reads an empty `Shortnames` result.
Hub B then completes `Add`, writing its hub launcher and the all-hubs launcher.
A resumes and deletes the all-hubs launcher at line 75.
Both calls succeed, leaving B's hub launcher without the all-hubs launcher D10 requires while any hub has one.
Independent dobbies call these operations without a desktop-wide lock.

### [MEDIUM] The child protocol is outside the tests

**Location:** internal/machinecli/start_test.go:130

**Issue:** Every start fixture replaces `beamStart` with decoded entries.
No test reaches the new process invocation, cwd and argument forwarding, JSON-line selection or envelope decoding.
The command's child boundary can therefore fail independently of every new start test.
This coverage gap is distinct from the decoded-outcome defect in F4.

### [MEDIUM] Window routing is not asserted

**Location:** internal/machinecli/start_test.go:149

**Issue:** The fixture records workspace paths and terminal directories but checks only their counts.
Its terminal callback discards title and executable.
Opening the prime directory for every beam or opening the wrong workspace would leave the tests green.
D10 requires each terminal to attach to the corresponding started worktree.

### [MEDIUM] Failure continuation and opener recovery are untested

**Location:** internal/machinecli/start_test.go:193

**Issue:** The failed-child row has one hub whose child fails wholly.
It does not prove that later hubs continue or that mixed child outcomes open windows only for successful worktrees.
No row reaches workspace-open failure or terminal-open failure.
Those new branches promise warning-only workspace failure and an attach command after terminal failure.

### [MEDIUM] Platform directory discovery is bypassed

**Location:** internal/desktoplauncher/locate_test.go:52

**Issue:** The selection test substitutes fixed values for both directory resolvers.
Parser rows do not exercise config-directory selection, file reading, missing-file fallback or read failure.
No test reaches the new PowerShell-to-wslpath query chain or its error and empty-answer branches.
These boundaries determine whether the launchers reach the desktop at all.

## Verification

Reviewed the committed range `80ead58445718f329a7b068da7e40c3b7e11f883..HEAD` and board entry T11 against D10.
All four lens reports are saved beside this report.
The changed package set built successfully for Linux and for `GOOS=windows GOARCH=amd64`.
No runtime Windows desktop check or new test run was performed.

The scope lens found no unrelated implementation changes.
The child uses `selfexec.LymBinary`, logs its start and completion, and waits synchronously.
The focused runproc question produces no separate finding:
the PATTERN background requires registration for detached or long-lived run processes, not every synchronous helper.
