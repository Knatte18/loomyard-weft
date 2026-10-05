---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D11 and D7
    summary: D7 drops DEFER and D11 drops re-arming on the premise that background tasks survive /compact, but only the /clear smoke run supports it and it is neither an Auto-mode assumption nor an Open risk
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D13
    summary: D13 queue lifecycle is unbounded where it matters; a recorded but dead orch strand, a long watcher outage and batten's episode state across a restart are not addressed
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D12, D13, D14
    summary: A notice is typed into the orch as a turn and carries child error text, but no Decision bounds its content (one line, truncation, no control characters) and the notice prefix is referenced in D14 yet never defined
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D5, D10 and Constraints
    summary: A config load error on a stale permission_mode prompt is stated, but not where it surfaces for an already-running hub restarting on an old orch.yaml, nor which of Load or LoadOrTemplate applies
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D2
    summary: D2 skips the re-read for a requested cycle without stating what that path lets through (a cycle on a small or fresh reading) and what bounds it
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D15 and Auto-mode assumptions
    summary: The assumption that the registry records a process start is unverified, the audited registry entry reads only pid and sessionId, and if it is false D15 silently degrades to today's behavior with its tests still passing
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D16
    summary: D16 records State.Stuck for a too-short pane but does not say when it clears, so a held queue and cycle have no stated release condition
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D12 and Acceptance criteria
    summary: The quiet condition does not say whether it applies while the child is parked off running, which overlaps the state-change notice; notice_quiet_min and the one-line compact stencil have no acceptance criterion
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:Auto-mode assumptions and Open risks
    summary: The first Auto-mode bullet is process narrative that misstates the operator decisions recorded in the Decisions, and the last Open risk is a landing action, not a risk
---

### [MEDIUM] Compaction-survival premise of background tasks is unrecorded

**Location:** `decision-record.md` D11 rationale, D7 rationale, Auto-mode assumptions, Open risks.
**Issue:** D11 states "they survive `/compact`" as fact.
`internal/orchengine/doc.go` records only that a background task survives `/clear` (the 2026-10-02 smoke run).
D7 builds the dropped DEFER on the same claim, and D11 builds the dropped re-arming on it.
If compaction does not carry a pending background completion, the compact cycle silently loses in-flight work and nothing in the record bounds it.
**Fix:** Add the premise to Auto-mode assumptions and the consequence to Open risks, and state in D11 and D7 that it is unverified for `/compact`.
Add a smoke assertion that a background task's completion still reaches the session after a live compact cycle.

### [MEDIUM] Notice queue lifecycle

**Location:** `decision-record.md` D13 (and D12 episode rule).
**Issue:** The queue is gated on "the prime's orch state records a strand", not on that strand being alive, so a stale or dead strand collects notices no watcher delivers.
D13's bound ("one line per episode per run") does not cover a watcher that is down for days, and no cap, expiry or drop rule exists.
D12 does not say whether episode state survives a batten restart; if it is in memory, a restart re-notifies every current condition.
**Fix:** State that the queue is capped (oldest dropped, with a log line) and that a notice for a strand that is gone is dropped on delivery or when the watcher starts.
State where episode state lives and that a restart does not re-notify an episode already notified.

### [MEDIUM] Notice content and prefix are unbounded and undefined

**Location:** `decision-record.md` D12 notice line, D13 typing, D14 start stencil.
**Issue:** A notice is typed into the orch session as a user turn and includes the child's error text, which can be arbitrary multi-line text produced by a failed run.
The Decision does not bound what a notice can make the orch do through typed text: no single-line rendering, no length cap, no stripping of control characters or a leading slash.
D14 refers to "the notice prefix" but no Decision defines it, so the plan writer would have to invent it, and the stencil rule and the writer could disagree.
**Fix:** In D12 define the prefix literal and render the notice as one line, with the error truncated to a stated length and stripped of control characters and newlines.
State that the typed notice can never begin with `/` and that the orch acts on it only through `lyx batten status`.

### [MEDIUM] Failure mode of a stale config on a running hub

**Location:** `decision-record.md` D5, D10, Constraints (PATTERN-config-strictness).
**Issue:** D10 makes `permission_mode: prompt` a load error.
A hub whose `orch.yaml` still carries it fails at the next restart of the watcher or the orch.
The record does not say where the error surfaces (start verb, watcher start, status), whether the running session is left alone, or whether the load is strict `Load` or the degrading `LoadOrTemplate`.
The Constraints line says only "keeps its current loader".
**Fix:** State the loader by name, that the error is raised at orch start and watcher start before any pane is touched, and that a running watcher is unaffected until it restarts.

### [LOW] Requested cycle skips the re-read without a stated bound

**Location:** `decision-record.md` D2.
**Issue:** "A requested cycle does not depend on the reading and is not re-read" lets a cycle through on a fresh or small reading, which in clear mode discards a small session.
The Decision does not say what bounds it.
**Fix:** State that a requested cycle is operator-initiated, still needs phase idle and the idle probe, skips only the threshold, and is not subject to the D8 hold.

### [LOW] Registry process-start assumption unverified

**Location:** `decision-record.md` D15, Auto-mode assumptions, Open risks.
**Issue:** `internal/shuttleengine/claudeengine/resume.go` unmarshals only `pid` and `sessionId`, so the record's process-start field is a guess about Claude's registry shape.
If the field is absent, D15 equals today's behavior and the PID-reuse item ships as a no-op, yet every listed test passes.
**Fix:** Add an Open risk saying so, and have D15 name the observable result when no start time is recorded (the adopt still refuses, with a message that says the pid could not be proven reused).

### [LOW] D16 stuck reason has no release condition

**Location:** `decision-record.md` D16.
**Issue:** The Decision sets `State.Stuck` but not when it clears.
Notices and cycles are held while the pane stays short; a status line that is never cleared would lie after a resize.
**Fix:** State that the reason is cleared on the next tick whose idle probe passes, and that notices keep queuing under D13's cap meanwhile.

### [LOW] Quiet condition overlap and missing criteria

**Location:** `decision-record.md` D12, Acceptance criteria.
**Issue:** The quiet notice reads "status.json unchanged for `notice_quiet_min` with the driver strand alive", with no restriction to a running child; an awaiting child would produce a state notice and later a quiet notice.
No criterion tests the `notice_quiet_min` config key or default, and none tests that the compact stencil renders to one line, although the Constraints require it.
**Fix:** Restrict the quiet condition to a running child, and add the two criteria.

### [NIT] Process narrative and a landing action in the record

**Location:** `decision-record.md` Auto-mode assumptions first bullet, Open risks last bullet.
**Issue:** "No operator answered; every pick below the operator's recorded decisions is an auto-pick" is process narrative the plan writer does not act on, and it sits oddly beside the three operator decisions.
The last Open risk is a board amendment due at landing, not a risk of the design.
Both carry no requirement the plan writer needs beyond the board amendment itself.
**Fix:** Drop the first bullet (the support log's Interview holds it) and move the board amendment to the Constraints as a landing obligation.

Pass over the rest of `decision-record.md` for relocation: the Rationale lines for D1, D3, D4, D6, D8, D11, D12 and D13 each carry a requirement or its reason and stay.
The Scope bullets map to D1 to D18 and each has a criterion apart from the two named in the last LOW finding.
