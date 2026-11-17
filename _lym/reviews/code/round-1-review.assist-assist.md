---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/beamcli/hubstop.go:112
    summary: The detached stop exits without stopping a hub whose prime has a subpath anchor
    fix: Set spawnDetachedStop's cmd.Dir to prime.AnchorPath() instead of prime.WorktreePath(), and cover a non-root anchor in the command launch test.
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/orchcli/stop.go:74
    summary: An in-session BeamDown can kill its caller before ending the dobby and cannot end its ancestor through the final call
    fix: Make the in-session BeamDown path hand shutdown to a worker outside the orch's process ancestry before removing the strand. Have that worker finish the preceding shutdown steps and end the registered orch process through runproc last. Preserve runproc's caller and ancestor protection, and add a controlled process/session regression covering invocation from inside the orch.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/orchcli/stop.go:74
    summary: The final process end substitutes EndRole for the EndSubject operation specified by D9
    fix: Wire BeamDown's final process end through runproc.EndSubject as D9 specifies, preserving its spared hub-service roles and the preceding shutdown order. If retaining the narrower EndRole operation is required, obtain a parent amendment to D9 before retaining that departure; the landing summary alone does not amend the decision.
    origin: lens:code-scope
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/beamcli/hubstop_test.go:63
    summary: The hub-stop tests bypass command dispatch, detached execution and production worktree selection
    fix: Add public-command cases for parent launch, detached child execution without another launch, and slug-plus-all refusal. Use a controlled hub fixture to cover selection of the prime, local beams and task pairs regardless of ownership and exercise DownWorktree against the supplied location. Include the anchored launch regression from F1 in these cases rather than adding a duplicate test.
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/beamcli/hubstop_test.go:28
    summary: Failure continuation is covered only for pair-down
    fix: Extend TestRunHubStop's existing table with pause, enumeration, BeamDown, reap and lift errors. Assert the subsequent operations and exact failure entries, including an owned pause returned alongside a pause error and a failed lift that must not report PauseLifted.
    origin: lens:code-tests
---

### [BLOCKING] Detached execution starts outside the configured anchor

**Location:** internal/beamcli/hubstop.go:112

**Issue:** The child starts in `prime.WorktreePath()` and re-enters `actingLocation`.
That calls strict `lymcwd.Resolve`, which requires the configured anchor directory.
For an anchor such as `src`, the parent returns a spawned PID while the child refuses before pausing or stopping anything.
The resolver contract is in `internal/lymcwd/lymcwd.go:75` and `pattern/PATTERN-cwd-resolution.md`.

### [BLOCKING] In-session shutdown does not survive its first step

**Location:** internal/orchcli/stop.go:74

**Issue:** The requested last-step termination must work when the orch invokes `lym beam stop` itself.
Before reaching the added end, `beamDown` removes the orch strand through `stopOrch`.
The production adapter reaches `reedengine.RemoveStrand`, which destroys the pane and reaps its descendants.
`internal/reedengine/lifecycle.go:1148` force-kills surviving pane children without sparing this caller.
The invocation can therefore die before ending its dobby.
If no tracked pane remains, the final `EndRole` still skips an orch provider that is the caller's ancestor.
That protection is explicit in `internal/runproc/end.go:213`.
Appending the call does not deliver D9's in-session guarantee.

### [MEDIUM] The process-end API departs from the decision

**Location:** internal/orchcli/stop.go:74

**Issue:** D9 names `runproc.EndSubject`, while `c.endOrch` is wired to `EndRole` in `internal/orchcli/cli.go:320`.
The landing summary acknowledges the substitution, but no amendment or qualifying departure settles it.
The review instructions expressly make D9 a yardstick alongside the Goal.
This finding concerns that unresolved scope departure, independently of F2's lifecycle defect.
`EndSubject` already spares the hub-service roles listed in `internal/runproc/end.go`.

### [MEDIUM] The new command and target selection have no behavior coverage

**Location:** internal/beamcli/hubstop_test.go:63

**Issue:** `TestRunHubStop` supplies every operation and its worktree list, then calls the private sequence directly.
It cannot detect broken flag dispatch, repeated spawning, a wrong child cwd, or omitted local worktrees.
No test executes `beam stop --all` or the new supplied-location `DownWorktree` entry point.
Existing reed-down tests cover the old CLI envelope and pause refusals, not these new entry paths.

### [MEDIUM] Other shutdown failures are untested

**Location:** internal/beamcli/hubstop_test.go:28

**Issue:** The table injects only a pair-down failure.
Pause, worktree enumeration, orch shutdown, reap and lift always succeed in the test.
The promised continuation after these errors and ownership cleanup after a partial pause have no regression assertion.
These cases belong in the existing sequence table.

## Review evidence and validation

Reviewed the committed range `80ead58445718f329a7b068da7e40c3b7e11f883..HEAD` against the Goal, amendments and D9.
All four lens reports are saved beside this report.
The shared pause extraction preserves carrier callbacks and ownership handling.
The reed-down extraction retains its existing refusal and envelope paths.
Both prime button templates carry the required command and confirmation.
The extra `hibernatecli` wiring is required to reuse the existing dependency construction.
No changed code touches `dobbyengine`, `machinecli` or `termwindow`.

The tier-0 gate invocation could not start because this sandbox cannot write the gate and runproc files under `_bolt/.lym`.
No test result is claimed from that invocation.
This environmental limit is not a finding against the diff.
