---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D9, A2, Open risks
    summary: D9 drops the skills step after a compaction with no Bound line, on premise A2, which the current orchengine doc contradicts and no open risk covers
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D4
    summary: D4 removes runs that are "otherwise not attachable", wider than the attach probe's respawn-eligible verdict, so its bound does not name the exact set removed
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D1, D2, A3
    summary: D1 does not say how the input box is matched to the sent text, so "never a draft of other text" is not testable and long or wrapped text may read as a draft
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D8, R3, Acceptance criteria
    summary: D8's "the turn end that made the watcher look" is undefined for a restarted watcher or several turn ends read at once, while the criteria split by transcript count
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D3
    summary: D3 skips a reflection with no Bound line and does not say what covers the uncovered halt note
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Added after Discussion, Constraints, R4
    summary: The ordering decision names both prunes but Constraints and R4 name only test-suite-prune-core, and the same fact is stated three times
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D6
    summary: D6 lets the plan writer replace the voucher mechanism when the cause shows it cannot work, with no bound on that redesign
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D11, Constraints
    summary: D11 does not say a new watcher clears the stopping record, and Constraints claim D11's envelope names a lyx verb that D11 never names
  - id: F9
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:Acceptance criteria
    summary: Several stated bounds have no acceptance scenario, namely D1 no Enter on a draft or running turn, D4 failed removal, D10 wait for the next idle probe, D11 past the wait bound
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:D1, D4, D10 Rationale
    summary: Rationale lines enumerate current callers and rows (D1, D4) and repeat a rejected alternative (D10); both are perishable or belong in the support log
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:D9, D10
    summary: D9 updates the orchengine doc.go reload section but D10 also changes that section and does not say so
---

### [MEDIUM] D9 removes a guard with no bound, on an unsupported premise

**Location:** `decision-record.md` D9, A2, Open risks.
**Issue:** D9 stops retyping skills after a compaction, which removes a guard: a session that lost its skills would now go on without them.
The attack-surface rule requires the Decision to say what that lets through and what bounds it.
D9 has no `Bound:` line; the bound is only implied by A2.
A2 rests on "as the board entry states", while `internal/orchengine/doc.go` today says "A clear, a compaction and an auto-compaction each lose the session's skills and role".
The record does not say what happens if A2 fails, and R1 covers D10 only.
**Fix:** Add a `Bound:` line to D9 stating that only the compaction entries skip the skills step, `/clear` keeps it, and that the pointer step still reads the role file.
Add an open risk for A2 failing: a session after a compaction then lacks a skill until its next `/clear` or restart, and the plan writer confirms A2 against the Claude Code version before removing the step.

### [MEDIUM] D4's bound is wider than the probe's verdict

**Location:** `decision-record.md` D4.
**Issue:** The decision removes strands of runs that reached "a terminal outcome or is otherwise not attachable".
The attach probe has three verdicts: attachable, respawn-eligible and error.
An error candidate makes the probe fail the start before any respawn, so removal is reached only when every matching candidate is respawn-eligible.
"Otherwise not attachable" reads as covering the error verdict as well, and it does not tie the removal to the probe's own decision.
**Fix:** State the removal set as the matching runs the attach probe judged respawn-eligible, and say that a start the probe refuses removes nothing.

### [MEDIUM] D1 leaves the input-box matching rule open

**Location:** `decision-record.md` D1, D2, A3.
**Issue:** The Bound says an extra Enter never lands on a draft of other text, but the Decision does not say how the box content is compared with the sent text.
D2's premise is that Claude Code reads a fast burst as a paste, and a rejected alternative in the support log records that a large paste collapses to a placeholder.
A long or wrapped sent text could then appear in the box as a placeholder or in wrapped lines, which a literal comparison reads as "not holding the sent text" and sends no Enter.
A3 only covers telling the box from transcript text.
**Fix:** Add to D1 the matching requirement: the box counts as holding the sent text when its content, with line wrapping and whitespace normalized, contains the sent text or a bounded leading part of it, and anything else, a placeholder included, counts as not holding it and falls back to the appearance-only check (R2).
Add a regression scenario for a placeholder box and for a draft of other text.

### [MEDIUM] D8's trigger turn end is not defined for every entry

**Location:** `decision-record.md` D8, R3, Acceptance criteria.
**Issue:** D8 defines freshness by "the turn end that made the watcher look".
The criteria define it by transcript: "followed by exactly one turn end" enters the sequence, "later turn ends" moves the baseline.
When a watcher restarts, or reads several turn ends in one poll, the turn end that made it look is not the same as the count of turn ends after the boundary, and R3 already names that gap.
**Fix:** Define the rule in transcript terms in D8: a boundary is fresh only when exactly one turn end follows it in the transcript read at that point, and that turn end is the one the watcher is reacting to; any boundary with more turn ends after it moves the baseline.
Align R3 and the criteria to the same wording.

### [LOW] D3 has no Bound line and leaves the note's coverage implicit

**Location:** `decision-record.md` D3.
**Issue:** D3 skips a reflection, which is a removed step.
The limits are in the Decision text, but no `Bound:` line names what is skipped and what covers the halt note afterwards.
**Fix:** Add a `Bound:` line: only the `escalation-to-human` kind skips, the note stays on disk unarchived, and the Friction-Reflect row that a resumed run reaches covers it.

### [LOW] The ordering decision and its restatements disagree

**Location:** `decision-record.md` "Added after Discussion", Constraints last bullet, R4.
**Issue:** The decision says both `test-suite-prune-core` and `test-suite-prune-rest` land first.
The Constraints bullet and R4 name only `test-suite-prune-core`.
The same ordering is stated in all three places.
**Fix:** Keep the ordering in the "Added after Discussion" decision, and reduce the Constraints bullet and R4 to a pointer to it, naming both prunes.

### [LOW] D6 can replace the voucher with no bound

**Location:** `decision-record.md` D6.
**Issue:** "The voucher mechanism stays the means ... unless the cause shows it cannot" lets the plan writer redesign the mechanism, and nothing bounds that redesign.
**Fix:** Add that a replacement must keep the Bound: a driver that dies mid-producer is still reported, and the acceptance criteria for D6 are the whole contract.

### [LOW] D11 leaves a stale stopping record and an unnamed way forward

**Location:** `decision-record.md` D11, Constraints.
**Issue:** The record does not say who clears the stopping record, so a stale one could make a later `start` wait while a healthy new watcher holds the lock.
Constraints say D11's envelope names a way forward that is a lyx verb, but D11 names none.
**Fix:** Say a watcher clears the stopping record when it starts, and name the verb in the envelope the way D4 names `lyx reed remove`.

### [LOW] Acceptance criteria miss several stated bounds

**Location:** `decision-record.md` Acceptance criteria.
**Issue:** No scenario covers D1's no extra Enter on a draft of other text or a running turn, D4's failed removal ending the start with the named error, D10's wait for the next passing idle probe, or D11's report of a live watcher past the wait bound.
**Fix:** Add one criterion for each, as a row of the nearest existing scenario where one exists.

### [NIT] Rationale lines enumerate and repeat

**Location:** `decision-record.md` D1, D4, D10 Rationale.
**Issue:** D1 lists the current delivery callers and D4 lists the current rows, which go stale when a caller or row changes.
D10's "which would cost the session id and the Remote Control link" repeats a rejected alternative that the support log already holds.
No requirement is lost by cutting them: the one-fix-covers-all-sends point stays.
**Fix:** Name the group instead of its members in D1 and D4, and cut the repeated clause from D10.

### [NIT] The doc.go update covers only D9

**Location:** `decision-record.md` D9, D10.
**Issue:** D9 says the orchengine "The reload sequence" section is updated, and D10 adds a step to the same section without saying so.
**Fix:** Say in D10 that the same section describes the `/reload-plugins` step.
