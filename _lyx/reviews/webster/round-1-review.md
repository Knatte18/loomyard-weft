---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/wait.go:339-349
    summary: boundaryHeld is never cleared when the writer leaves the boundary while covered, so a later uncovered gated Done is evaluated twice
    fix: In Run.Wait, clear run.boundaryHeld wherever run.gateAtBoundary is set false (wait.go:527, wait.go:560 and send.go:858), and clear it in the arrival branch at wait.go:336 before it sets gateAtBoundary true. Add a gate_test.go row where a gated Done arrives while covered, the writer starts a new turn before the pause lifts, a second gated Done arrives uncovered, and the boundary is evaluated exactly once.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/proc/procgroup_linux.go (configurePlatformGroupTerminate, cmd.Cancel)
    summary: A SIGTERM to the group that fails with an error other than ESRCH returns the error with no kill fallback, where the group-kill path falls back to killing the command alone
    fix: In the cmd.Cancel closure, when termErr is non-nil and not ESRCH, return cmd.Process.Kill() after report so the shell cannot outlive the cancel; keep returning termErr for ESRCH. Update the ConfigureGroupTerminate doc line "a failed SIGTERM is returned from the cancel and starts no SIGKILL timer" in procgroup.go to match.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/proc/procgroup_linux.go (configurePlatformGroupTerminate)
    summary: The failed-SIGTERM branch and the awaitGone early return for a cancel that never ran have no test; the success path is pinned only through the verifyrun integration tier
    fix: Add a Linux test file in internal/proc with rows for a cancel whose SIGTERM fails (no timer armed, awaitGone returns at once, and after F2 the shell is killed) and for awaitGone called without a cancel. Reuse the F2 fix's behavior as the expected result.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: PATTERN.md (PATTERN-review-round, PATTERN-fabric-vocabulary, PATTERN-documentation-lifecycle lines)
    summary: The overview trims eleven entry lines where card 8 names eight, and the overview ends 150 bytes shorter where the plan states 21
    fix: Either restore the three clauses cut beyond card 8 (review-round ", each finding fixed or disputed, end on a verdict", fabric-vocabulary "; warp and weft only to tell its sides apart", documentation-lifecycle "; a built one lives in its package's `doc.go`") up to the byte room the headroom lint allows, or leave the cuts and record in the commit message of the fix that they are intended. Their backgrounds already state each rule, so no background edit is needed.
---

### [MEDIUM] Held boundary flag survives a turn change

**Location:** internal/shuttleengine/wait.go:339-349
**Issue:** `boundaryHeld` is set when a gated Done arrives while a pause covers the session, and cleared only in the branch that fires once the pause lifts with the writer still at the boundary. The code that sets `gateAtBoundary` false (`wait.go:527`, `wait.go:560`, `send.go:858`) leaves it set. If the writer starts a new turn while covered and a later gated Done arrives uncovered, the arrival branch evaluates the boundary and sets `gateAtBoundary` true. On the next tick the stale `boundaryHeld && !covered && gateAtBoundary` branch fires and evaluates the same boundary again, which can re-send a re-prompt or re-read a verdict twice. I read the branches and did not run this sequence.

### [LOW] Failed group SIGTERM leaves the shell alive

**Location:** internal/proc/procgroup_linux.go (cmd.Cancel in configurePlatformGroupTerminate)
**Issue:** On a SIGTERM failure other than ESRCH, such as EPERM, the cancel returns the error, arms no SIGKILL timer and does not kill the command alone. `awaitGone` then returns at once because `terminated` is false. The group-kill path falls back to `cmd.Process.Kill()` on the same kind of failure. The case is unlikely for a group the process started itself, hence LOW.

### [LOW] Group-terminate branches untested at the proc surface

**Location:** internal/proc/procgroup_linux.go (configurePlatformGroupTerminate)
**Issue:** The diff adds no test file under `internal/proc`. The integration rows in `internal/verifyrun/run_integration_test.go` pin SIGTERM-then-SIGKILL, but nothing pins the failed-SIGTERM return or `awaitGone` without a cancel.

### [LOW] Overview trims beyond card 8

**Location:** PATTERN.md
**Issue:** Card 8 lists eight entries to shorten. The diff also shortens `PATTERN-review-round`, `PATTERN-fabric-vocabulary` and `PATTERN-documentation-lifecycle`; the card-9 commit message says this was to keep the headroom lint green per commit. The rules survive in `pattern/PATTERN-review-round.md:3`, `pattern/PATTERN-fabric-vocabulary.md:4` and `pattern/PATTERN-documentation-lifecycle.md:3`, so no rule is lost, but the overview's text for entries outside the declared list changed. No `Pattern-Departure:` trailer applies; this is a plan departure, not a PATTERN one.

## Rejected

- `PATTERN-verified-tree` declared text not landed (code-pattern-fit asked, no finding): the line matches the declared text character for character, and the background and tripwire carry card 8's additions.
- `PATTERN-pause-holds-prompts` landing: the summary line in `PATTERN.md` and the declared text in full in the background file match the plan's Shared Decisions departure.
- code-affected-set MEDIUM that two trimmed rules live nowhere: rejected, the backgrounds for review-round and fabric-vocabulary state them (see F4), and the other eight cuts are already in their backgrounds, as card 8 permits.
- code-tests LOW that `internal/orchcli/pausegeometry_test.go` is a weak source scan: rejected, its comment states the wired runner is unreachable from a test, so the scan is the available pin and no behavior test is possible.
- code-tests unread files (`gate_test.go`, `run_test.go`, `resume_test.go`, `batchpause_test.go`, `verify_integration_test.go`, `delivery_test.go`): no duplicate named, so no test-economy finding.
- Commit trailers: no `Pattern-Departure:` trailer exists in the range, so none names an unknown entry.
