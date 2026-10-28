---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3 and D5
    summary: The LLM driver session decides re-launch and its binary watch only by the hand_back mark, so after D3 drops that mark a llm-driven loom or darn run re-launches the halted escalate row on a new build, the re-step D3 closes for batten, and the record is silent on it
    fix: In D3, add the LLM driver as a reader of the escalate halt and state its treatment beside batten's, for example that the driver stencil's re-launch and binary-watch rules skip a stop whose reason opens with the escalate prefix as they skip one marked hand_back, so only the arbiter's resume or the orch's lym loom resume moves it. Add contracts/stencils/shed/shed-template-driver.md and contracts/stencils/shed/shed-template-driver-guide.md to D5's docs list, and add an acceptance criterion that the driver stencil names the escalate halt among the stops it never re-launches.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, the driver bullet
    summary: The criterion says a driver that starts at a halted escalate stop builds it from the status and sends it to the arbiter, but a starting driver either reuses an open attempt's envelope or steps the blocked row, which re-runs Finalize or Parent-Align, so the start half pins a path that does not exist and contradicts D4's own Bound
    fix: Replace the clause "a driver that starts or re-checks at a halted escalate stop, built from the status, also sends it to the arbiter with no tackler call" with "a driver whose re-check finds the run moved to a halted escalate stop sends that status-built stop, read by its Error, to the arbiter with no tackler call; a driver restarted with an open arbiter attempt for the stop reuses that attempt; a driver starting at a halted escalate stop with no open attempt re-steps the row, and only a re-escalation reaches the arbiter".
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3, the batten re-step sentence
    summary: D3 says a new build never re-runs the row over an undecided collision and only the arbiter's or the orch's resume moves it, but the driver's own build hand-over re-steps the escalate stop once, as D4's Bound states, so D3's guarantee is broader than the design delivers
    fix: In D3, scope the sentence to batten, reading "so batten's re-step never re-runs the row over an undecided collision on a new build; the driver's own build hand-over re-steps it once, per D4's Bound", and keep D4's Bound as it is.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5
    summary: contracts/specs/loom-status-spec.md's hand_back section defines the key for loom and darn as a stop opening with the landing-stop prefix and says every driver hands it back, which goes stale once D3's escalate prefix is no hand-back, and D5 does not list the spec
    fix: Add to D5's list a bullet for contracts/specs/loom-status-spec.md's hand_back section, saying a loom or darn fixer escalate halt opens with the escalate prefix, is not marked hand_back, and goes to the arbiter first.
  - id: F5
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md, ### Turn 4 — burler round 1 fixes
    summary: The round-1 fix turn sits as an H3 under the Question ledger, after its bullets, while the spec makes the Interview the turn-by-turn section and the ledger a list of questions
    fix: Move the whole Turn 4 block from under ## Question ledger to the end of ## Interview, after Turn 3, unchanged.
---

### [MEDIUM] The LLM driver re-launches a halted escalate on a new build

**Location:** `_lym/discussion/decision-record.md` D3 and D5

**Issue:** `contracts/stencils/shed/shed-template-driver.md:98` lets the driver session re-launch a `blocked` stop that is "not marked `hand_back`" when `binary_changed`, line 101 exempts only a stop marked `hand_back`, and line 145 starts the binary watch at a hand-back not marked `hand_back`.
`internal/shedrun/seed.go:44-50` keeps `DriverLLM` a legal loom driver, and `internal/loomcli/driverprompt.go` fills this stencil.
D3 drops the `hand_back` mark from a loom or darn escalate halt on purpose, and closes the resulting re-step for batten's binary re-step alone, citing that "a new build never re-runs the row over an undecided collision".
The LLM driver path breaks that purpose in the same way and the record neither names it nor lists its stencil and guide in D5.

### [LOW] The driver acceptance criterion names a start path that never builds a status stop

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria, the driver bullet

**Issue:** `goDriver.run` (`internal/sheddriver/drive.go`) first tries `resumeOpenAttempt`, which rebuilds the stop from the attempt's saved envelope, so `Reason` carries the halt.
Otherwise `checkBlockedStart` builds a status stop only to hand back on a `StartCheck` refusal, deciding nothing, and the driver then calls `Step`.
A blocked status does not short-circuit in `internal/shedengine/run.go:221-223`: the current producer is called again, so Finalize re-merges and a fresh fixer runs on the turn the escalate gave back.
Only `recheck`'s moved stop is a status-built stop that reaches `decideDrive`, which D4's Reason-or-Error rule does cover.
The criterion's start half therefore asks the plan for a test of an unreachable path, and it contradicts D4's Bound, which says a new driver re-steps the stop first.
Interactions, bound-breaker and walkthrough each raised this, and the handler found it too; merged here.

### [LOW] D3's re-step guarantee is wider than the driver delivers

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** `decideDrive`'s hand-over row (`internal/sheddriver/driverules.go`) precedes the arbiter-first row, and the successor driver steps the blocked status, re-running the row's gate and a new fixer over the undecided collision with no arbiter or orch resume.
D4's Bound states this one re-step, so D3's "a new build never re-runs the row …; only the arbiter's resume or the orch's `lym loom resume` moves it" contradicts the record's own D4.

### [LOW] The loom status spec's hand_back contract is missing from D5

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** `contracts/specs/loom-status-spec.md:96-97` defines `hand_back` for loom and darn by the landing-stop prefix at Parent-Align and Finalize and says every driver hands such a stop back.
Under D3 a loom or darn escalate halt is no longer such a stop, so the status contract needs the change in the same commit, per CLAUDE.md's docs rule, and D5 does not name it.

### [NIT] Turn 4 sits under the question ledger

**Location:** `_lym/discussion/support-log.md`, `### Turn 4 — burler round 1 fixes`

**Issue:** The loom discussion spec makes `## Interview` the turn-by-turn section and `## Question ledger` the list of questions.
The round-1 fix turn is appended after the ledger's bullets, so a reader of the Interview misses the turn that reshaped D3 and D4.

## Rejected

- A prefix-collision constraint on the escalate prefix (a new prefix that is itself a prefix of `LandingStopPrefix` would make D3's predicate match every landing halt): every reader of the predicate, the driver table, `arbiterstop.Check` and batten's re-step, applies `HandsBack` first, so no reader misclassifies a stop, and D3's "so neither `arbiterstop.HandsBack` nor its decision-halt check matches it" already binds the other direction.
- The orch role stencil's line naming the `hand_back` mark on a landing halt: D5 already rewrites that stencil's `Landing halts` section, and naming the exact line is call-site enumeration the rubric leaves to Plan.

## Verdicts on the focus directives

- Field the seam reads: D4 names `Reason` for an envelope-built stop and `Error` for a status-built stop, which is right on every path that decides one; the acceptance criterion's start clause is F2.
- Pawl outcome: it now rests on `Deps.EscalateJudgedFirst` false keeping `LandingStopPrefix`, so `HandsBack` hands it back by mechanism, and the pawl acceptance criterion pins it; the refusal it cites holds at `internal/loomcli/decision.go`'s missing-record branch of the append. Not a defect.
- Prefix readers and the loop's spend: every reader classifies the prefix as intended (see Rejected), and D4's Bound states that the escalate-resume loop spends the run's arbiter spawn budget (`arbiter_max_spawns`, 3 by default per run, shared with every other arbiter-eligible stop), after which later stops hand back. Not a defect.
