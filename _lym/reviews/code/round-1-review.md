---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/wait.go:881-903
    summary: A gated waiting turn end keeps waitingTasks after a turn start, so the bound can count Done while the agent is mid-turn reading the shell result
    fix: "In pollEventsTick, when turnStarts holds a turn start past run.waitingTurnEnd.offset, clear run.waitingTasks and run.waitingTurnEnd (moving the tasks to run.countedTasks only if the record needs them) before the len(data)==0 and len(events)==0 returns call expiredTurnEnd, so the gated expiry cannot fire in a turn already under way. Add a test row in waiting_test.go that appends a turn start after the waiting line, advances the clock past the bound, and expects no Done until the next turn end."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/wait.go:918
    summary: The turnStartAfter guard that skips queueing the shell nudge has no test
    fix: "Add a row or assertion in waiting_test.go (a variant of TestWait_GatedShellNudgeNamesEachShellOnce) where a turn-start signal follows the waiting turn end in the same batch, and assert reed.SendTextCalls stays empty."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/wait.go:1186-1201
    summary: The pause drop and the failed-send path of sendShellNudge have no test
    fix: "Add two small variants of TestWait_GatedShellNudgeNamesEachShellOnce in waiting_test.go: one with the pause record covering the run, asserting no SendText call, and one with a send that fails, asserting no retry and that the run still reaches Done at the bound."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/wait.go:1160-1201
    summary: Shells are marked named in queueShellNudge before the send, so a pause drop or a failed send leaves the agent never told about them
    fix: "Keep the IDs queueShellNudge names in a pending slice beside run.shellNudge, and mark them in run.nudgedShells only after sendWithin succeeds in sendShellNudge. A dropped or failed send then leaves them unnamed, and the next waiting turn end over them queues the line again."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/waitmark.go:175-177
    summary: The shellWaitActive doc says a gated run's shell-only waiting turn end is judged at once, which the diff made false
    fix: "Reword the last sentence of the shellWaitActive doc comment: a gated run with every output file present counts Done in the tick the last shell crosses the bound, so the list shows only while shells are outstanding at staggered first-seen times; drop the claim that a gated run's turn end is judged at once."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/spec.go:124
    summary: The AwaitedShellPrefixes comment says an awaited shell keeps a gated waiting turn end from being judged as a plain turn end, which no longer describes a non-awaited shell with every output file present
    fix: "Reword spec.go line 124 to say an awaited shell keeps a gated waiting turn end waiting with no nudge and no expiry, and keeps an ungated wait from expiring."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/runlevel.go:39
    summary: The masterAwaitedShellPrefixes comment says a gated turn end waiting only on non-awaited shells is judged as a plain turn end, which holds only with an output file missing
    fix: "Change the clause in runlevel.go line 39 to say a gated turn end waiting only on non-awaited shells is held when an output file is missing and is not counted done inside background_shell_wait_min."
---

### [MEDIUM] Gated expiry can count Done mid-turn on a stale waiting list

**Location:** `internal/shuttleengine/wait.go:881-903`, `internal/shuttleengine/wait.go:1112-1130`
**Issue:** The diff stops clearing `waitingTasks` and `waitingTurnEnd` on the gated branch, so they stay set across ticks until a plain turn end or a new waiting turn end replaces them.
A turn start carries no event, so a tick that reads one returns through `expiredTurnEnd` with the list intact.
Once the first-seen stamp is past `background_shell_wait_min`, the gated branch returns `OutcomeDone` while the agent is in the turn the shell's completion or the nudge started.
Before the diff a gated shell-only waiting turn end finished at once, so this window did not exist.
The only guard is `boundaryIdle`, which applies to engines with the `SessionCycler` idle reading, and the summary's claim that the run "goes on to its next turn end" does not hold for the other engines.
Class `design`: the change does not do what the entry asks, a run that waits for the shell and reads its result.

### [MEDIUM] The nudge's turn-start guard is untested

**Location:** `internal/shuttleengine/wait.go:918`
**Issue:** The branch added to skip queueing when a turn start is already read past the waiting turn end has no row.
Removing the guard fails no test, since `TestWait_GatedShellNudgeNamesEachShellOnce` carries no turn-start signal.

### [MEDIUM] The nudge's pause drop and failed send are untested

**Location:** `internal/shuttleengine/wait.go:1186-1201`
**Issue:** `sendShellNudge` adds two behaviours, the drop under `run.covered` and the logged, unretried failure.
No test in `waiting_test.go` reaches either.

### [LOW] A dropped or failed nudge leaves its shells marked as named

**Location:** `internal/shuttleengine/wait.go:1160-1201`
**Issue:** `queueShellNudge` writes `nudgedShells` before `sendShellNudge` sends.
A pause drop or a failed send therefore leaves the shells marked, and no later waiting turn end over them queues the line.
The run then reaches Done at the bound with the agent never told, which is the outcome the change exists to prevent.
The `sendShellNudge` doc says a failed send is not retried because the shell's completion starts the next turn, which does not cover the pause drop.

### [LOW] `shellWaitActive` doc is stale

**Location:** `internal/shuttleengine/waitmark.go:175-177`
**Issue:** The doc says a gated run's shell-only waiting turn end "is judged at once, so neither leaves a list to show".
A gated run with every output file present now waits on such a list up to the bound, and `syncShellWait` reads `shellWaitActive` every tick.

### [LOW] `AwaitedShellPrefixes` doc is stale

**Location:** `internal/shuttleengine/spec.go:124`
**Issue:** The comment contrasts an awaited shell with a "plain turn end" judgment.
A non-awaited shell at a gated turn end with every output file present is no longer judged as a plain turn end.

### [LOW] `masterAwaitedShellPrefixes` doc is stale

**Location:** `internal/websterengine/runlevel.go:39`
**Issue:** The comment says a gated turn end waiting only on non-awaited shells "is judged as a plain turn end".
That holds only with an output file missing; `internal/websterengine/doc.go`, which the diff updated, says so.

## Rejected

- The `master-fork-running/case.yaml:12` wording (lens:code-affected-set F4): the following clause of the same sentence says "with Master's output files missing", so the sentence is already qualified, and the fixture's outcomes are correct.
- No nudge asserted for a gated run with an output file missing or with a fork outstanding (lens:code-tests, third finding): the nudge is queued only inside the files-present shell-only branch, and an absence assertion on each existing row would add no behaviour coverage.
- The gate-slot claim in `_lym/landing/summary.md`: borne out, since `TestAcquire_HolderProcessDeathFreesSlot` at `internal/gateslot/pool_integration_test.go:16` kills a holder process and asserts the slot frees.
- Scope against the Goal: the diff takes the entry's first option (refuse Done while a shell runs and tell the agent), bounded by `background_shell_wait_min`; no change falls outside what the entry licenses, and no PATTERN entry is violated or departed from.
- Focus questions on the ungated refactor, `shellFirstSeen` stamping before `expiredTurnEnd`, the staggered two-shell case and the `sendWithin` locking: the forks ran each and found them correct.
