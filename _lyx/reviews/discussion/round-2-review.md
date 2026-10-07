---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:15
    summary: Scope and the D3 heading promise the whole process tree dies on expiry, but D3 delivers only the shell's process group on Unix and the shell alone on Windows
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:132
    summary: The Constraint places process-group setup in verifyrun/internal/proc, but verifyrun imports only the standard library and internal/logger and internal/proc has no process-group helper
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:56
    summary: The 60-minute verify deadline is a bare constant with no stated test seam, so the expiry criterion cannot run in the test tiers
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:6
    summary: Goal and Scope say reed never revives a finished strand, but D5 relaunches a gated run's strand whether its gate passed or not, and D5's Bound labels that case gate unpassed
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:44
    summary: Only a gone or replaced strand is stated as a logged no-op for the landing callback; no failure of the set or clear op, and nothing on shuttle's own sets and clears, is bounded against failing the verify or the gate wait
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:95
    summary: D7 takes the bootstrap lock but does not say that the run state is read after the lock is held, whether the acquisition waits or refuses, or that every path releases it
  - id: F7
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:98
    summary: D7's refusals for a missing status file, a done run and an awaiting run name no way forward, while Constraints require every new D7 refusal to name a lyx verb
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:101
    summary: D7 reuses start's merge-in-progress and not-parked refusal kinds and the shared resume branch, whose messages tell the caller to re-run lyx loom start, the verb D7 exists to replace
  - id: F9
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:104
    summary: A halted run whose run lock is still held falls into the no-live-driver refusal with a batten or start way forward, though start would no-op there; the merge row's precedence over the not-parked row is also unstated
  - id: F10
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D6's revive resumes every non-live strand of the pair, not only the driver, and its Bound does not say what else that relaunches or how a Go-driven child with no driver strand is told apart from a dead LLM driver
---

### [MEDIUM] Scope and the D3 heading overpromise the kill on Windows and for escaped descendants

**Location:** `_lyx/discussion/decision-record.md:15`, `_lyx/discussion/decision-record.md:51`
**Issue:** Scope says the verify command is "killed ... with the whole process tree killed on expiry", and D3's heading says "its whole process tree dies on expiry".
The body of D3 delivers less: on Unix a kill of the shell's own process group, so a descendant that moved to its own group or session survives (D3's Bound says as much), and on Windows `os.Process.Kill` on the shell alone, so descendants outlive the deadline (D3 says so too).
`verifyrun` may import only the standard library and `internal/logger`, so no Job Object or process-table walk is available to close that gap.
The heading and the Scope bullet are what the plan writer reads first, and they promise a tree kill on every platform.
D3 also says the Windows kill "logs that the kill was degraded", which reads as if `verifyrun` detected a surviving descendant; it cannot, so the log line is on every Windows deadline or cancel kill and says only that the shell alone was killed.
**Fix:** Reword the Scope bullet and the D3 heading to the Unix process-group kill and the Windows shell-only kill, and state that the Windows log line is emitted on every kill there, naming the shell-only limit.

### [MEDIUM] The process-group Constraint names a package `verifyrun` may not import

**Location:** `_lyx/discussion/decision-record.md:132`
**Issue:** The Constraint says process-group setup "stays in `verifyrun`/`internal/proc`".
`internal/verifyrun`'s package doc says it imports only the standard library and `internal/logger`, and D3's own rejected-alternative reasoning relies on that limit (support log, Windows Job Object entry).
`internal/proc` today has no process-group helper: its Unix `Detach` sets `Setsid` (a new session), and its Windows `Detach` sets `CREATE_NEW_PROCESS_GROUP` plus `CREATE_NO_WINDOW`, neither of which D3 asks for.
The plan writer cannot tell whether to add an import to `verifyrun`, widen its stated import limit, or keep the setup inside `verifyrun`.
**Fix:** Say the process-group setup and the group kill live in `verifyrun` itself, in per-OS files using only the standard library's `syscall`, and drop `internal/proc` from the Constraint.

### [LOW] The 60-minute verify deadline has no test seam

**Location:** `_lyx/discussion/decision-record.md:56`
**Issue:** D3 fixes the deadline as a 60-minute named constant, and an Acceptance criterion requires a test of an expired deadline ending as a failed gate.
The Constraints bullet on `PATTERN-test-speed` covers only the elapsed-time clock, and that pattern requires a production timeout to be injectable so a test shortens it instead of waiting it out.
As written, the only way to meet the criterion in the default tier is not stated.
**Fix:** Add to the Constraints bullet that the verify deadline is a parameter a test shortens, with the 60-minute constant as the production value.

### [MEDIUM] Goal and Scope say "never revives a finished strand", D5 does not deliver that for gated runs

**Location:** `_lyx/discussion/decision-record.md:6`, `_lyx/discussion/decision-record.md:16`, `_lyx/discussion/decision-record.md:76`
**Issue:** Goal says "reed never revives a shuttle strand whose run is finished", and Scope says `reed resume` "drops, rather than revives, a strand whose work is done (#409)".
D5 gives a done-when list to an ungated run only and relaunches every strand without one, so a gated run's strand is relaunched after its gate has passed as well as while it is unpassed.
D5's Bound words the relaunched case as "is gated (gate unpassed)", which suggests a passed gate is dropped; reed cannot know a gate passed, so it is not.
The Decision does state what is dropped (an ungated run's non-live strand with every output present) and what still re-prompts (a gated run's live agent after a failed gate), but the Goal and Scope promise more than that, and the residual (a finished gated run's strand relaunched when shuttle's own teardown did not run) is stated nowhere.
**Fix:** Narrow the Goal and Scope sentences to an ungated run's strand, reword D5's Bound to "a gated run's strand, whatever its gate's state", and add the finished-gated relaunch as an Open risk with what bounds it: reed relaunches the session without typing any prompt, and shuttle's next attach or teardown retires it.

### [MEDIUM] The wait-mark failure bound covers one case of one caller

**Location:** `_lyx/discussion/decision-record.md:44`
**Issue:** D2 says "a callback whose strand is gone or replaced is a logged no-op and never fails the verify", and the Acceptance criterion repeats it for the landing callback only.
Setting or clearing the mark runs a tmux command and can fail for other reasons (a tmux error, the server gone).
For shuttle's own sets and clears on gate entries and the background-shell wait, the record states no failure bound at all.
A set or clear error returned from inside `Wait` would be a mechanism-failure return in a loop whose negative answers `PATTERN-completion-signal` audits, and would turn a display-only mark into a failed gate or a failed run.
D2's own Bound says the mark is display only and never read by a Go decision, which only holds if no failure of it changes an outcome.
**Fix:** State in D2 that any failure of the mark op, at a shuttle site or in the landing callback, is logged and changes no verdict, error return or gate outcome, and make the Acceptance criterion cover both callers.

### [MEDIUM] D7 names the bootstrap lock but not how it is taken or when state is read

**Location:** `_lyx/discussion/decision-record.md:95`
**Issue:** D7 says the verb "takes the same bootstrap lock `lyx loom start` takes, so the two never run at once".
`lyx loom start` acquires it with a blocking write lock and holds it across the spawn and the run-lock handshake.
The record does not say whether `lyx loom resume` also waits for it or refuses while it is held, and it does not say the run state, the driver strand's liveness and the park marker are read after the lock is held.
A state read taken before the wait would let a resume that queued behind a mid-spawn start refuse with "no live driver" for a driver that start has just brought up.
Every failure path of `start` releases the lock explicitly, and the record does not carry that over.
**Fix:** State in D7 that the verb waits for the lock like `start`, reads every input of the outcome table only after holding it, and releases it on every path out.

### [MEDIUM] Three D7 refusals have no stated way forward

**Location:** `_lyx/discussion/decision-record.md:98`
**Issue:** D7 refuses a missing status file or a done run with "nothing to resume", and an awaiting run with "awaiting is resumed by batten on a recorded decision".
The Constraints bullet on `PATTERN-refusal-way-forward` requires every new D7 refusal to name a lyx verb as its way forward, and the refusal spec defines a way forward as a verb, a transition the run takes itself, or a git command.
For a missing status file the way forward is `lyx loom start`, and for an awaiting run it is `lyx loom approve` or `lyx loom reject` followed by batten's resume; a done run has none of the three, so either the refusal is not one (a no-op success saying the run is done) or its way forward must be chosen.
The plan writer would otherwise invent these, and a spec row and a test are due for each.
**Fix:** Name the way forward of each of the three outcomes in D7, and decide whether a done run is a refusal or a no-op success.

### [LOW] D7's reused refusals and branch carry messages that name `lyx loom start`

**Location:** `_lyx/discussion/decision-record.md:101`
**Issue:** D7 refuses over an unfinished merge "with start's existing merge-in-progress kind" and a not-yet-parked driver with "start's existing refusal kind", and shares the parked-driver branch with `start`.
The messages those code paths write today tell the caller to re-run `lyx loom start` (the merge refusal, the not-parked refusal, and the resume-send failure), which is the verb the orch must stop using for a halted child.
Reusing the kind is right; reusing the message sends the orch back to the verb D7 replaces.
**Fix:** State that the shared branch takes the verb name for its way-forward text from its caller, so the messages `lyx loom resume` produces name `lyx loom resume`.

### [LOW] D7 leaves a halted run with a held run lock, and the merge-versus-not-parked order, to the plan writer

**Location:** `_lyx/discussion/decision-record.md:104`
**Issue:** A halted status can be persisted while a Go driver still holds the run lock for post-run bookkeeping.
D7 handles a held run lock only for a running run; a halted run with the lock held and no driver strand falls into the "no live driver strand" refusal, whose way forward (`lyx batten run <slug>`, or `lyx loom start --no-attach`) is wrong while the lock is held, since `start` treats a held lock as nothing to spawn.
The outcomes also overlap where a merge is unfinished and the driver is live but not parked: the merge row is listed first, but `start` never probes the merge for a live driver without a park marker, so the order is a choice D7 does not state.
**Fix:** Add the halted-with-lock-held case to D7 with its outcome and way forward (retry after the lock holder finishes, named through `lyx loom status`), and state which of the merge and not-parked refusals wins when both apply.

### [MEDIUM] D6's revive resumes the whole pair and does not distinguish a Go-driven child

**Location:** `_lyx/discussion/decision-record.md:83`
**Issue:** D6 revives a halted or awaiting child's driver "through reed resume" of the pair.
Reed resume relaunches every non-live, non-hidden strand of the worktree session, not only the driver; D5 drops only ungated finished strands, so a halted step's gated agent strand and any operator strand also come back.
D6's Bound lists what the revive does not do (no shed step, no marker removed, no resume line) but not what else it starts, which is the attack-surface question for a new routing edge.
The trigger is "driver strand not live", and a Go-driven child never has a driver strand (the seed's go arm spawns a detached process), while batten must not read the seed's driver (`PATTERN-driver-choice-single-site`).
Every halt of such a child would then spend its once-per-episode revive, warn that the driver is still not live, and name `lyx loom start --no-attach` in the notice.
**Fix:** State in D6 what else the pair-wide resume relaunches and what bounds it (relaunched sessions receive no prompt), and state how the revive tells a Go-driven child from a dead LLM driver without reading the seed's driver, or that it does not and the warn is accepted for it.

## Focus departures

- Directive 3 reads "Check the batten verb Decision that resumes a halted run".
  The record has no batten resume verb: the support log's Rejected alternatives records `lyx batten resume <slug>` as not chosen, and the verb that resumes a halted run is `lyx loom resume` (D7).
  I applied the directive's question, one outcome per run state plus the unfinished merge and the bootstrap lock, to D7, and read D6 (batten's restore on `lyx batten run`) for the batten side.
  F6 to F9 come from D7, and F10 from D6.
